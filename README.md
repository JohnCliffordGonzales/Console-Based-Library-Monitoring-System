import json
import os

DATA_FILE = "library_data.json"

books = [
    {
        "id": "B001",
        "title": "Introduction to Computing",
        "author": "John Smith",
        "quantity": 3
    },
    {
        "id": "B002",
        "title": "Python Programming",
        "author": "James Brown",
        "quantity": 2
    }
]

borrow_records = []


# SAVE DATA
def save_data():
    data = {
        "books": books,
        "borrow_records": borrow_records
    }

    try:
        with open(DATA_FILE, "w") as file:
            json.dump(data, file, indent=4)
    except OSError:
        print("Error: Unable to save library data.")


# LOAD DATA
def load_data():
    global books, borrow_records

    if os.path.exists(DATA_FILE):
        try:
            with open(DATA_FILE, "r") as file:
                data = json.load(file)

            books = data.get("books", [])
            borrow_records = data.get("borrow_records", [])

        except (json.JSONDecodeError, OSError):
            print("Warning: Could not load saved data.")
            print("Please check your library data file.")


# ADD BOOK
def add_book():
    print("\n--- ADD BOOK ---")

    # Validate book ID
    while True:
        book_id = input("Enter book ID: ").strip().upper()

        if not book_id:
            print("Book ID cannot be empty. Try again.")
            continue

        if any(book["id"].upper() == book_id for book in books):
            print("Book ID already exists. Enter another ID.")
            continue

        break

    # Validate title
    while True:
        title = input("Enter book title: ").strip()

        if not title:
            print("Title cannot be empty. Try again.")
        else:
            break

    # Validate author
    while True:
        author = input("Enter author: ").strip()

        if not author:
            print("Author cannot be empty. Try again.")
        else:
            break

    # Validate quantity without restarting the process
    while True:
        try:
            quantity = int(input("Enter quantity: "))

            if quantity < 0:
                print("Quantity cannot be negative. Try again.")
                continue

            break

        except ValueError:
            print("Invalid quantity. Please enter a whole number.")

    books.append({
        "id": book_id,
        "title": title,
        "author": author,
        "quantity": quantity
    })

    save_data()
    print("Book added and saved successfully.")


# VIEW BOOKS
def view_books():
    print("\n--- ALL BOOKS ---")

    if not books:
        print("No books available.")
        return

    for book in books:
        print("-------------------------")
        print("Book ID:", book["id"])
        print("Title:", book["title"])
        print("Author:", book["author"])
        print("Available Copies:", book["quantity"])


# SEARCH BOOK
def search_book():
    print("\n--- SEARCH BOOK ---")

    search = input("Enter book title or author: ").strip().lower()

    if not search:
        print("Search cannot be empty.")
        return

    found = False

    for book in books:
        if (search in book["title"].lower()
                or search in book["author"].lower()):

            print("-------------------------")
            print("Book ID:", book["id"])
            print("Title:", book["title"])
            print("Author:", book["author"])
            print("Available Copies:", book["quantity"])
            found = True

    if not found:
        print("No matching book found.")


# UPDATE BOOK
def update_book():
    print("\n--- UPDATE BOOK ---")

    book_id = input("Enter book ID to update: ").strip().upper()

    for book in books:
        if book["id"].upper() == book_id:
            while True:
                title = input("Enter new title: ").strip()
                if title:
                    break
                print("Title cannot be empty. Try again.")

            while True:
                author = input("Enter new author: ").strip()
                if author:
                    break
                print("Author cannot be empty. Try again.")

            borrowed = sum(
                1 for record in borrow_records
                if record["book_id"] == book["id"]
            )

            while True:
                try:
                    quantity = int(input("Enter new quantity: "))

                    if quantity < borrowed:
                        print(
                            "Quantity cannot be less than "
                            "the number of borrowed copies."
                        )
                        continue

                    if quantity < 0:
                        print("Quantity cannot be negative.")
                        continue

                    break

                except ValueError:
                    print("Enter a valid whole number.")

            book["title"] = title
            book["author"] = author
            book["quantity"] = quantity

            save_data()
            print("Book updated and saved successfully.")
            return

    print("Book not found.")


# BORROW BOOK
def borrow_book(borrower_type):
    print("\n--- BORROW BOOK ---")

    borrower_id = input(
        f"Enter {borrower_type} ID: "
    ).strip()

    if not borrower_id:
        print("Borrower ID cannot be empty.")
        return

    borrower_name = input("Enter full name: ").strip()

    if not borrower_name:
        print("Borrower name cannot be empty.")
        return

    book_id = input("Enter book ID: ").strip().upper()

    for book in books:
        if book["id"].upper() == book_id:
            if book["quantity"] <= 0:
                print("Book is currently unavailable.")
                return

            book["quantity"] -= 1

            borrow_records.append({
                "book_id": book["id"],
                "borrower_id": borrower_id,
                "borrower": borrower_name,
                "borrower_type": borrower_type
            })

            save_data()
            print("Book borrowed and record saved successfully.")
            return

    print("Book not found.")


# RETURN BOOK
def return_book(borrower_type):
    print("\n--- RETURN BOOK ---")

    borrower_id = input(
        f"Enter {borrower_type} ID: "
    ).strip()

    book_id = input("Enter book ID: ").strip().upper()

    for record in borrow_records:
        if (
            record["borrower_id"].lower() == borrower_id.lower()
            and record["book_id"].upper() == book_id
            and record["borrower_type"] == borrower_type
        ):
            for book in books:
                if book["id"] == record["book_id"]:
                    book["quantity"] += 1
                    break

            borrow_records.remove(record)

            save_data()
            print("Book returned successfully.")
            return

    print("Borrowing record not found.")


# VIEW BORROWING RECORDS
def view_borrow_records():
    print("\n--- BORROWING RECORDS ---")

    if not borrow_records:
        print("No borrowing records.")
        return

    for record in borrow_records:
        print("-------------------------")
        print("Book ID:", record["book_id"])
        print("Borrower Type:", record["borrower_type"])
        print("Borrower ID:", record["borrower_id"])
        print("Borrower Name:", record["borrower"])


# LIBRARIAN MENU
def librarian_menu():
    while True:
        print("\n================================")
        print("        LIBRARIAN MENU")
        print("================================")
        print("1. Add Book")
        print("2. View Books")
        print("3. Search Book")
        print("4. Update Book")
        print("5. View Borrowing Records")
        print("6. Logout")

        choice = input("Enter your choice: ").strip()

        if choice == "1":
            add_book()
        elif choice == "2":
            view_books()
        elif choice == "3":
            search_book()
        elif choice == "4":
            update_book()
        elif choice == "5":
            view_borrow_records()
        elif choice == "6":
            print("Logging out...")
            break
        else:
            print("Invalid choice. Please try again.")


# STUDENT AND TEACHER MENUS
def borrower_menu(borrower_type):
    while True:
        print("\n================================")
        print(f"         {borrower_type.upper()} MENU")
        print("================================")
        print("1. View Books")
        print("2. Search Book")
        print("3. Borrow Book")
        print("4. Return Book")
        print("5. Logout")

        choice = input("Enter your choice: ").strip()

        if choice == "1":
            view_books()
        elif choice == "2":
            search_book()
        elif choice == "3":
            borrow_book(borrower_type)
        elif choice == "4":
            return_book(borrower_type)
        elif choice == "5":
            print("Logging out...")
            break
        else:
            print("Invalid choice. Please try again.")


# MAIN MENU
def main():
    load_data()

    while True:
        print("\n================================")
        print(" CONSOLE-BASED LIBRARY")
        print("       MONITORING SYSTEM")
        print("================================")
        print("1. Librarian")
        print("2. Student")
        print("3. Teacher")
        print("4. Exit")

        choice = input("Enter your choice: ").strip()

        if choice == "1":
            librarian_menu()
        elif choice == "2":
            borrower_menu("Student")
        elif choice == "3":
            borrower_menu("Teacher")
        elif choice == "4":
            save_data()
            print(
                "Thank you for using the Library Monitoring System."
            )
            break
        else:
            print("Invalid choice. Please try again.")


main()
