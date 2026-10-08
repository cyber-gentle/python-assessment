# Learn2Earn Inventory Management Assessment

---

## A1. Source Code

### File: `assess.py`

```python
import sys


class learn2earn:
    def __init__(self):
        self.resources = [
            {"id": "R001", "name": "Laptop", "category": "Electronics", "total": 10, "available": 10},
            {"id": "R002", "name": "Keyboard", "category": "Accessories", "total": 5, "available": 5},
            {"id": "R003", "name": "Headset", "category": "Accessories", "total": 3, "available": 3},
        ]

        self.fellows = {"F001": "Ada", "F002": "John", "F003": "Grace"}
        self.borrow_records = []

    def add_item(self, name, category, quantity, resource_id=""):
        if not name.strip():
            return "Please enter a resource name."
        if not category.strip():
            return "Please enter a category."
        if not isinstance(quantity, int) and quantity <= 0:
            return "Quantity must be a positive whole number."

        if resource_id == "":
            highest_id = 0
            for item in self.resources:
                number = int(item["id"][1:])
                if number > highest_id:
                    highest_id = number
            resource_id = f"R{highest_id + 1:03d}"
        else:
            resource_id = resource_id.strip().upper()

        for item in self.resources:
            if item["id"] == resource_id:
                return "That resource ID already exists."

        new_item = {
            "id": resource_id,
            "name": name.strip(),
            "category": category.strip(),
            "total": quantity,
            "available": quantity,
        }
        self.resources.append(new_item)
        return f"Resource added successfully. ID: {resource_id}"

    def list_resources(self):
        return self.resources

    def lend_item(self, fellow_id, resource_id, quantity):
        fellow_id = fellow_id.strip().upper()
        resource_id = resource_id.strip().upper()

        if fellow_id not in self.fellows:
            return "Fellow ID was not found."
        if not isinstance(quantity, int) or isinstance(quantity, bool) or quantity <= 0:
            return "Quantity must be a positive whole number."

        found_item = None
        for item in self.resources:
            if item["id"] == resource_id:
                found_item = item
                break

        if found_item is None:
            return "Resource ID was not found."
        if quantity > found_item["available"]:
            return f"Not enough stock. Only {found_item['available']} available."

        found_item["available"] = found_item["available"] - quantity
        new_record = {
            "fellow_id": fellow_id,
            "resource_id": resource_id,
            "quantity": quantity,
            "returned": 0,
        }
        self.borrow_records.append(new_record)
        return f"{self.fellows[fellow_id]} borrowed {quantity} {found_item['name']}(s)."

    def return_item(self, fellow_id, resource_id, quantity):
        fellow_id = fellow_id.strip().upper()
        resource_id = resource_id.strip().upper()

        if fellow_id not in self.fellows:
            return "Fellow ID was not found."
        if not isinstance(quantity, int) or isinstance(quantity, bool) or quantity <= 0:
            return "Quantity must be a positive whole number."

        amount_on_loan = 0
        for record in self.borrow_records:
            if record["fellow_id"] == fellow_id and record["resource_id"] == resource_id:
                amount_on_loan += record["quantity"] - record["returned"]

        if amount_on_loan == 0:
            return "This fellow has not borrowed that resource."
        if quantity > amount_on_loan:
            return f"Return rejected. This fellow only has {amount_on_loan} on loan."

        amount_left_to_return = quantity
        for record in self.borrow_records:
            if record["fellow_id"] == fellow_id and record["resource_id"] == resource_id:
                still_borrowed = record["quantity"] - record["returned"]
                if still_borrowed > 0:
                    if still_borrowed >= amount_left_to_return:
                        record["returned"] = record["returned"] + amount_left_to_return
                        amount_left_to_return = 0
                    else:
                        record["returned"] = record["returned"] + still_borrowed
                        amount_left_to_return = amount_left_to_return - still_borrowed

                if amount_left_to_return == 0:
                    break

        for item in self.resources:
            if item["id"] == resource_id:
                item["available"] = item["available"] + quantity
                return f"{self.fellows[fellow_id]} returned {quantity} {item['name']}(s)."

        return "Resource ID was not found."

    def search_resources(self, name):
        results = []
        search_text = name.lower()
        for item in self.resources:
            if search_text in item["name"].lower():
                results.append(item)
        return results

    def filter_by_category(self, category):
        results = []
        category = category.lower()
        for item in self.resources:
            if item["category"].lower() == category:
                results.append(item)
        return results

    def reports(self):
        total_units = 0
        available_units = 0
        borrowed_units = 0
        borrowed_by_resource = {}

        for item in self.resources:
            total_units = total_units + item["total"]
            available_units = available_units + item["available"]
            borrowed_by_resource[item["id"]] = 0

        for record in self.borrow_records:
            still_borrowed = record["quantity"] - record["returned"]
            borrowed_units = borrowed_units + still_borrowed
            borrowed_by_resource[record["resource_id"]] = (
                borrowed_by_resource[record["resource_id"]] + still_borrowed
            )

        low_stock = []
        for item in self.resources:
            if item["available"] < 3:
                low_stock.append(item)

        most_borrowed_count = 0
        for resource_id in borrowed_by_resource:
            if borrowed_by_resource[resource_id] > most_borrowed_count:
                most_borrowed_count = borrowed_by_resource[resource_id]

        most_borrowed = []
        if most_borrowed_count > 0:
            for item in self.resources:
                if borrowed_by_resource[item["id"]] == most_borrowed_count:
                    most_borrowed.append({
                        "id": item["id"],
                        "name": item["name"],
                        "borrowed": most_borrowed_count,
                    })

        return {
            "total_units": total_units,
            "available_units": available_units,
            "borrowed_units": borrowed_units,
            "low_stock": low_stock,
            "most_borrowed": most_borrowed,
        }


def show_resources(resources):
    if len(resources) == 0:
        print("No matching resources found.")
        return

    for item in resources:
        print(f"{item['id']} | {item['name']} | {item['category']}")
        print(f"  Total: {item['total']}, available: {item['available']}")


def show_report(app):
    report = app.reports()
    print(f"Total units: {report['total_units']}")
    print(f"Available units: {report['available_units']}")
    print(f"Currently borrowed: {report['borrowed_units']}")

    print("Resources with fewer than 3 units available:")
    if len(report["low_stock"]) == 0:
        print("  None")
    else:
        for item in report["low_stock"]:
            print(f"  {item['name']}: {item['available']} available")

    print("Resource(s) with the most units borrowed:")
    if len(report["most_borrowed"]) == 0:
        print("  None")
    else:
        for item in report["most_borrowed"]:
            print(f"  {item['name']}: {item['borrowed']} borrowed")


def run_demo():
    app = learn2earn()

    print("1. F001 borrows 2 laptops")
    print(app.lend_item("F001", "R001", 2))
    print(f"Laptop available: {app.resources[0]['available']}\n")

    print("2. F002 borrows 3 keyboards")
    print(app.lend_item("F002", "R002", 3))
    print(f"Keyboard available: {app.resources[1]['available']}\n")

    print("3. F001 returns 1 laptop")
    print(app.return_item("F001", "R001", 1))
    print(f"Laptop available: {app.resources[0]['available']}\n")

    print("4. F003 requests 4 headsets")
    print(app.lend_item("F003", "R003", 4))
    print(f"Headset available: {app.resources[2]['available']}\n")

    print("5. F002 tries to return 4 keyboards")
    print(app.return_item("F002", "R002", 4))
    print(f"Keyboard available: {app.resources[1]['available']}\n")

    print("6. Search for LAPtop")
    show_resources(app.search_resources("LAPtop"))

    print("\n7. Inventory report")
    show_report(app)


def run_menu():
    app = learn2earn()

    while True:
        print("\nLearn2Earn Inventory")
        print("1. List resources")
        print("2. Add a resource")
        print("3. Lend a resource")
        print("4. Return a resource")
        print("5. Search by name")
        print("6. Filter by category")
        print("7. Show report")
        print("0. Exit")

        choice = input("Choose an option: ")

        if choice == "0":
            print("Goodbye.")
            break
        elif choice == "1":
            show_resources(app.list_resources())
        elif choice == "2":
            name = input("Resource name: ")
            category = input("Category: ")
            quantity_text = input("Quantity: ")
            resource_id = ""
            try:
                quantity = int(quantity_text)
                print(app.add_item(name, category, quantity, resource_id))
            except ValueError:
                print("Please enter quantity as a whole number.")
        elif choice == "3":
            fellow_id = input("Fellow ID: ")
            resource_id = input("Resource ID: ")
            quantity_text = input("Quantity to lend: ")
            try:
                quantity = int(quantity_text)
                print(app.lend_item(fellow_id, resource_id, quantity))
            except ValueError:
                print("Please enter quantity as a whole number.")
        elif choice == "4":
            fellow_id = input("Fellow ID: ")
            resource_id = input("Resource ID: ")
            quantity_text = input("Quantity to return: ")
            try:
                quantity = int(quantity_text)
                print(app.return_item(fellow_id, resource_id, quantity))
            except ValueError:
                print("Please enter quantity as a whole number.")
        elif choice == "5":
            name = input("Name to search: ")
            show_resources(app.search_resources(name))
        elif choice == "6":
            category = input("Category to filter by: ")
            show_resources(app.filter_by_category(category))
        elif choice == "7":
            show_report(app)
        else:
            print("Please select one of the menu options.")


if __name__ == "__main__":
    if len(sys.argv) > 1 and sys.argv[1] == "--demo":
        run_demo()
    else:
        run_menu()
```

---

## A2. Demonstration Output and Test Evidence

### Actual Terminal Run Output (Steps 1–7)
Executed via: `python3 assess.py --demo`

```text
1. F001 borrows 2 laptops
Ada borrowed 2 Laptop(s).
Laptop available: 8

2. F002 borrows 3 keyboards
John borrowed 3 Keyboard(s).
Keyboard available: 2

3. F001 returns 1 laptop
Ada returned 1 Laptop(s).
Laptop available: 9

4. F003 requests 4 headsets
Not enough stock. Only 3 available.
Headset available: 3

5. F002 tries to return 4 keyboards
Return rejected. This fellow only has 3 on loan.
Keyboard available: 2

6. Search for LAPtop
R001 | Laptop | Electronics
  Total: 10, available: 9

7. Inventory report
Total units: 18
Available units: 14
Currently borrowed: 4
Resources with fewer than 3 units available:
  Keyboard: 2 available
Resource(s) with the most units borrowed:
  Keyboard: 3 borrowed
```

### Additional Invalid-Input Tests (Actual Run Output)

#### 1. Invalid Fellow ID Test
* **Action:** Attempting to lend an item to an unregistered Fellow ID (`F999`).
* **Actual Output:**
  ```text
  Fellow ID was not found.
  ```

#### 2. Negative/Non-Positive Quantity Test
* **Action:** Attempting to borrow a negative quantity (`-3`).
* **Actual Output:**
  ```text
  Quantity must be a positive whole number.
  ```

#### 3. Non-Existent Resource ID Test
* **Action:** Attempting to lend an unrecognised Resource ID (`R999`).
* **Actual Output:**
  ```text
  Resource ID was not found.
  ```

#### 4. Returning Unborrowed Item Test
* **Action:** Fellow `F003` attempting to return `R001` (Laptop) which they never borrowed.
* **Actual Output:**
  ```text
  This fellow has not borrowed that resource.
  ```

---

## A3. Project Design Explanation

### 1. Named Functions and Methods
1. **`add_item(name, category, quantity, resource_id="")`**: Validates inputs, handles automatic incremental ID generation (e.g., `R004`) if omitted, and appends a newly initialised resource record into inventory with total and available counts.
2. **`lend_item(fellow_id, resource_id, quantity)`**: Validates fellow ID existence, resource ID validity, and verifies that current available stock is sufficient before decrementing available inventory and logging a new entry in `borrow_records`.
3. **`return_item(fellow_id, resource_id, quantity)`**: Ensures the fellow actually has active loans for the given resource, rejects excessive returns, updates the `returned` count across loan records, and restores the available quantity back to stock.
4. **`reports()`**: Computes aggregate inventory analytics across all items and borrow records, calculating total units, available units, currently borrowed units, low-stock alerts ($<3$ units), and resource(s) with the highest borrowed volume.

*(Other supporting functions include: `list_resources()`, `search_resources(name)`, `filter_by_category(category)`, `show_resources(resources)`, and `show_report(app)`.)*

### 2. Representation of Inventory and Fellow Loans
* **Inventory Representation:**
  * Stored in `self.resources` as a **list of dictionaries** (`list[dict]`).
  * Each dictionary represents a resource with 5 keys:
    * `"id"` (`str`): Unique identifier (e.g., `"R001"`).
    * `"name"` (`str`): Human-readable item name (e.g., `"Laptop"`).
    * `"category"` (`str`): Classification group (e.g., `"Electronics"`).
    * `"total"` (`int`): Total owned inventory quantity.
    * `"available"` (`int`): Remaining units currently in stock to lend.
* **Fellows Representation:**
  * Stored in `self.fellows` as a **dictionary** (`dict[str, str]`) mapping Fellow IDs to names (e.g., `{"F001": "Ada", "F002": "John", "F003": "Grace"}`).
* **Fellow Loans Representation:**
  * Stored in `self.borrow_records` as a **list of dictionaries** (`list[dict]`).
  * Each record tracks a loan transaction with:
    * `"fellow_id"` (`str`): ID of the fellow who borrowed the item.
    * `"resource_id"` (`str`): ID of the borrowed resource.
    * `"quantity"` (`int`): Units borrowed during this transaction.
    * `"returned"` (`int`): Units returned against this record so far.
  * Active units currently on loan are calculated dynamically as `record["quantity"] - record["returned"]`.

### 3. Design Limitation
* **Primary Limitation — In-Memory Storage / Lack of Data Persistence:**
  * All program state (`self.resources`, `self.fellows`, and `self.borrow_records`) exists exclusively in transient Python memory during script execution.
  * Once the program terminates or the CLI exits, all newly created resources, borrow histories, and inventory adjustments are permanently lost, resetting the system back to the initial default state. To make this production-ready, data must be persisted to a relational database (e.g., SQLite/PostgreSQL) or flat files (e.g., JSON/CSV).
* **Additional Limitations:**
  * **Static Fellows:** Fellow accounts cannot be created or edited via the application interface.
  * **$O(n)$ Search Complexity:** Items and borrow logs use sequential lists requiring full linear scans for lookups instead of indexed key-value lookups.
  * **No Timestamps / Due Dates:** Records lack transaction timestamps or return deadlines, preventing tracking of overdue assets.
