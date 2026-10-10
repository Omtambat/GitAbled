
import csv
import os

CSV_FILE = "library_books.csv"


class Book:
    def __init__(self, title, author, copies=1, borrowed=0):
        self.title = title.strip()
        self.author = author.strip()
        self.__copies = int(copies)
        self.__borrowed = int(borrowed)

    @property
    def available(self):
        return self.__copies

    @property
    def borrowed(self):
        return self.__borrowed

    def add_copies(self, copies):
        if copies > 0:
            self.__copies += copies

    def borrow(self):
        if self.__copies > 0:
            self.__copies -= 1
            self.__borrowed += 1
            return True
        return False

    def return_book(self):
        if self.__borrowed > 0:
            self.__borrowed -= 1
            self.__copies += 1
            return True
        return False

    def __str__(self):
        return (
            f"Title: {self.title} | Author: {self.author} | "
            f"Available: {self.__copies} | Borrowed: {self.__borrowed}"
        )


class Library:
    def __init__(self, name):
        self.name = name
        self.books = []

    def add_book(self, title, author, copies):
        # Find an existing title in this library
        for book in self.books:
            if book.title.lower() == title.lower():
                book.add_copies(copies)
                print("Existing book found. Copies added.")
                return

        # Create a new book if the title does not exist
        self.books.append(Book(title, author, copies))
        print("New book added successfully.")

    def show_books(self):
        print(f"\n--- {self.name} ---")

        if not self.books:
            print("No books in this library.")
            return

        for book in self.books:
            print(book)


class LibraryManager:
    def __init__(self):
        self.libraries = {}
        self.load_from_csv()

    def add_library(self, name):
        key = name.strip().lower()

        if not key:
            print("Library name cannot be empty.")
        elif key in self.libraries:
            print("This library already exists.")
        else:
            self.libraries[key] = Library(name.strip())
            self.save_to_csv()
            print("Library added successfully.")

    def get_library(self, name):
        return self.libraries.get(name.strip().lower())

    def save_to_csv(self):
        with open(CSV_FILE, "w", newline="", encoding="utf-8") as file:
            writer = csv.writer(file)

            writer.writerow([
                "library", "title", "author", "available", "borrowed"
            ])

            for library in self.libraries.values():
                if not library.books:
                    # Preserve libraries even when they have no books
                    writer.writerow([library.name, "", "", "", ""])
                else:
                    for book in library.books:
                        writer.writerow([
                            library.name,
                            book.title,
                            book.author,
                            book.available,
                            book.borrowed
                        ])

        print("Data saved to CSV successfully.")

    def load_from_csv(self):
        if not os.path.exists(CSV_FILE):
            return

        with open(CSV_FILE, "r", newline="", encoding="utf-8") as file:
            reader = csv.DictReader(file)

            for row in reader:
                name = row.get("library", "").strip()
                title = row.get("title", "").strip()

                if not name:
                    continue

                key = name.lower()

                if key not in self.libraries:
                    self.libraries[key] = Library(name)

                if title:
                    book = Book(
                        title,
                        row.get("author", ""),
                        int(row.get("available") or 0),
                        int(row.get("borrowed") or 0)
                    )
                    self.libraries[key].books.append(book)

    def show_libraries(self):
        if not self.libraries:
            print("No libraries available.")
            return

        print("\nAvailable Libraries:")
        for library in self.libraries.values():
            print("-", library.name)

    def view_all_entries(self):
        if not self.libraries:
            print("No library entries found.")
            return

        for library in self.libraries.values():
            library.show_books()


def get_positive_integer(message):
    while True:
        try:
            number = int(input(message))
            if number > 0:
                return number
            print("Please enter a number greater than zero.")
        except ValueError:
            print("Please enter a valid whole number.")


def main():
    manager = LibraryManager()

    while True:
        print("\n====== LIBRARY MANAGEMENT SYSTEM ======")
        print("1. Add library")
        print("2. Add book")
        print("3. View all entries")
        print("4. View one library")
        print("5. Borrow book")
        print("6. Return book")
        print("7. Save to CSV")
        print("8. Exit")

        choice = input("Enter your choice: ").strip()

        if choice == "1":
            name = input("Enter library name: ").strip()
            manager.add_library(name)

        elif choice == "2":
            name = input("Enter library name: ").strip()
            library = manager.get_library(name)

            if library is None:
                print("Library not found. Add the library first.")
                continue

            title = input("Enter book title: ").strip()
            author = input("Enter author name: ").strip()

            if not title or not author:
                print("Title and author cannot be empty.")
                continue

            copies = get_positive_integer("Enter number of copies: ")
            library.add_book(title, author, copies)
            manager.save_to_csv()

        elif choice == "3":
            manager.view_all_entries()

        elif choice == "4":
            name = input("Enter library name: ").strip()
            library = manager.get_library(name)

            if library:
                library.show_books()
            else:
                print("Library not found.")

        elif choice == "5":
            name = input("Enter library name: ").strip()
            library = manager.get_library(name)

            if library is None:
                print("Library not found.")
                continue

            title = input("Enter book title to borrow: ").strip()
            book = next(
                (b for b in library.books
                 if b.title.lower() == title.lower()),
                None
            )

            if book is None:
                print("Book not found.")
            elif book.borrow():
                manager.save_to_csv()
                print("Book borrowed successfully.")
            else:
                print("No copies available.")

        elif choice == "6":
            name = input("Enter library name: ").strip()
            library = manager.get_library(name)

            if library is None:
                print("Library not found.")
                continue

            title = input("Enter book title to return: ").strip()
            book = next(
                (b for b in library.books
                 if b.title.lower() == title.lower()),
                None
            )

            if book is None:
                print("Book not found.")
            elif book.return_book():
                manager.save_to_csv()
                print("Book returned successfully.")
            else:
                print("No borrowed copy exists to return.")

        elif choice == "7":
            manager.save_to_csv()

        elif choice == "8":
            manager.save_to_csv()
            print("Project closed. All data saved.")
            break

        else:
            print("Invalid choice. Please select 1 to 8.")


if __name__ == "__main__":
    main()
