#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Structure to represent a book [cite: 1, 2]
struct Book {
    int id; // Unique ID for each book [cite: 2]
    char title[100]; // The title of the book [cite: 2]
    char author[100]; // The author's name [cite: 2]
    int year; // The year of publication [cite: 2]
    struct Book *next; // A pointer to the next book to build a linked list [cite: 2]
};

struct Book *head = NULL; // Global head of the linked list

// Function to add a new book to the list [cite: 2, 4]
void addBook() {
    struct Book *newBook = (struct Book *)malloc(sizeof(struct Book));
    if (newBook == NULL) {
        printf("Memory allocation failed!\n");
        return;
    }

    printf("Enter book ID: ");
    scanf("%d", &newBook->id);
    getchar(); // Consume the newline character

    printf("Enter book title: ");
    fgets(newBook->title, sizeof(newBook->title), stdin);
    newBook->title[strcspn(newBook->title, "\n")] = 0; // Remove trailing newline

    printf("Enter author name: ");
    fgets(newBook->author, sizeof(newBook->author), stdin);
    newBook->author[strcspn(newBook->author, "\n")] = 0; // Remove trailing newline

    printf("Enter publication year: ");
    scanf("%d", &newBook->year);
    getchar(); // Consume the newline character

    newBook->next = NULL;

    if (head == NULL) {
        head = newBook;
    } else {
        struct Book *current = head;
        while (current->next != NULL) {
            current = current->next;
        }
        current->next = newBook;
    }
    printf("Book added successfully!\n");
}

// Function to display all books in the list [cite: 2, 4]
void displayBooks() {
    if (head == NULL) {
        printf("The library is empty.\n");
        return;
    }

    struct Book *current = head;
    printf("\n--- Current Books in Library ---\n");
    while (current != NULL) {
        printf("ID: %d\n", current->id);
        printf("Title: %s\n", current->title);
        printf("Author: %s\n", current->author);
        printf("Year: %d\n", current->year);
        printf("------------------------------\n");
        current = current->next;
    }
}

// Function to search for a book by title or ID [cite: 2, 4]
void searchBook() {
    if (head == NULL) {
        printf("The library is empty. Nothing to search.\n");
        return;
    }

    int choice;
    printf("Search by:\n1. ID\n2. Title\nEnter your choice: ");
    scanf("%d", &choice);
    getchar(); // Consume the newline character

    struct Book *current = head;
    int found = 0;

    if (choice == 1) {
        int searchId;
        printf("Enter ID to search: ");
        scanf("%d", &searchId);
        getchar(); // Consume the newline character

        while (current != NULL) {
            if (current->id == searchId) {
                printf("\n--- Book Found (by ID) ---\n");
                printf("ID: %d\n", current->id);
                printf("Title: %s\n", current->title);
                printf("Author: %s\n", current->author);
                printf("Year: %d\n", current->year);
                printf("--------------------------\n");
                found = 1;
                break;
            }
            current = current->next;
        }
    } else if (choice == 2) {
        char searchTitle[100];
        printf("Enter title to search: ");
        fgets(searchTitle, sizeof(searchTitle), stdin);
        searchTitle[strcspn(searchTitle, "\n")] = 0; // Remove trailing newline

        while (current != NULL) {
            // Case-insensitive comparison could be added here for a robust search
            if (strcmp(current->title, searchTitle) == 0) {
                printf("\n--- Book Found (by Title) ---\n");
                printf("ID: %d\n", current->id);
                printf("Title: %s\n", current->title);
                printf("Author: %s\n", current->author);
                printf("Year: %d\n", current->year);
                printf("-----------------------------\n");
                found = 1;
                // No break here, in case multiple books have the same title (though ID is unique)
            }
            current = current->next;
        }
    } else {
        printf("Invalid choice.\n");
        return;
    }

    if (!found) {
        printf("Book not found.\n");
    }
}

// Function to delete a specific book from the list [cite: 2, 4]
void deleteBook() {
    if (head == NULL) {
        printf("The library is empty. Nothing to delete.\n");
        return;
    }

    int deleteId;
    printf("Enter the ID of the book to delete: ");
    scanf("%d", &deleteId);
    getchar(); // Consume the newline character

    struct Book *current = head;
    struct Book *prev = NULL;
    int found = 0;

    while (current != NULL) {
        if (current->id == deleteId) {
            if (prev == NULL) { // Deleting the head node
                head = current->next;
            } else {
                prev->next = current->next;
            }
            free(current);
            printf("Book with ID %d deleted successfully.\n", deleteId);
            found = 1;
            break;
        }
        prev = current;
        current = current->next;
    }

    if (!found) {
        printf("Book with ID %d not found.\n", deleteId);
    }
}

// Function to save all book data to a file when the program exits [cite: 2, 4]
void saveToFile() {
    FILE *file = fopen("library.dat", "wb"); // Open in binary write mode
    if (file == NULL) {
        printf("Error opening file for saving.\n");
        return;
    }

    struct Book *current = head;
    while (current != NULL) {
        // Write each book's data directly to the file
        fwrite(&(current->id), sizeof(int), 1, file);
        fwrite(current->title, sizeof(char), 100, file);
        fwrite(current->author, sizeof(char), 100, file);
        fwrite(&(current->year), sizeof(int), 1, file);
        current = current->next;
    }

    fclose(file);
    printf("Library data saved to library.dat\n");
}

// Function to load book data from a file when the program starts [cite: 3, 4]
void loadFromFile() {
    FILE *file = fopen("library.dat", "rb"); // Open in binary read mode
    if (file == NULL) {
        printf("No existing library data found or error opening file for loading.\n");
        return;
    }

    // Clear any existing books in memory before loading
    struct Book *current = head;
    struct Book *temp;
    while (current != NULL) {
        temp = current;
        current = current->next;
        free(temp);
    }
    head = NULL;

    struct Book tempBook;
    while (fread(&(tempBook.id), sizeof(int), 1, file) == 1 &&
           fread(tempBook.title, sizeof(char), 100, file) == 1 &&
           fread(tempBook.author, sizeof(char), 100, file) == 1 &&
           fread(&(tempBook.year), sizeof(int), 1, file) == 1) {

        struct Book *newBook = (struct Book *)malloc(sizeof(struct Book));
        if (newBook == NULL) {
            printf("Memory allocation failed during loading!\n");
            fclose(file);
            return;
        }
        *newBook = tempBook; // Copy the read data
        newBook->next = NULL;

        if (head == NULL) {
            head = newBook;
        } else {
            struct Book *current_list = head;
            while (current_list->next != NULL) {
                current_list = current_list->next;
            }
            current_list->next = newBook;
        }
    }

    fclose(file);
    printf("Library data loaded from library.dat\n");
}

// Function to free all allocated memory before exiting
void freeList() {
    struct Book *current = head;
    struct Book *next;
    while (current != NULL) {
        next = current->next;
        free(current);
        current = next;
    }
    head = NULL;
}

int main() {
    loadFromFile(); // Load data when the program starts [cite: 3]

    int choice;
    do {
        printf("\n--- Mini Library Management System ---\n");
        printf("1. Add New Book\n"); // [cite: 4]
        printf("2. Display All Books\n"); // [cite: 4]
        printf("3. Search Book\n"); // [cite: 4]
        printf("4. Delete Book\n"); // [cite: 4]
        printf("5. Save and Exit\n"); // [cite: 4]
        printf("Enter your choice: ");
        scanf("%d", &choice);
        getchar(); // Consume the newline character

        switch (choice) {
            case 1:
                addBook();
                break;
            case 2:
                displayBooks();
                break;
            case 3:
                searchBook();
                break;
            case 4:
                deleteBook();
                break;
            case 5:
                saveToFile(); // Save data when the program exits [cite: 2, 4]
                freeList(); // Free allocated memory
                printf("Exiting program. Goodbye!\n");
                break;
            default:
                printf("Invalid choice. Please try again.\n");
        }
    } while (choice != 5);

    return 0;
}
