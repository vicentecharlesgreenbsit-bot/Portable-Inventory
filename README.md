Portable_Inventory.cpp
#include <iostream>
#include <fstream>
#include <iomanip>
#include <string>
using namespace std;

struct Item {
    int id;
    char name[50];
    int quantity;
    float price;
};

void displayHeader(const string& title) {
    cout << "\n=====================================\n";
    cout << "      " << title << endl;
    cout << "=====================================\n";
}

void addItem() {
    ofstream outFile("inventory.dat", ios::binary | ios::app);
    if (!outFile) {
        cout << "Error opening file!\n";
        return;
    }

    Item newItem;
    displayHeader("Add New Item");

    cout << "Enter Item ID: ";
    cin >> newItem.id;
    cin.ignore();
    cout << "Enter Item Name: ";
    cin.getline(newItem.name, 50);
    cout << "Enter Quantity: ";
    cin >> newItem.quantity;
    cout << "Enter Price: ";
    cin >> newItem.price;

    if (newItem.quantity < 0 || newItem.price < 0) {
        cout << "Quantity and Price must be non-negative.\n";
        return;
    }

    outFile.write(reinterpret_cast<char*>(&newItem), sizeof(Item));
    outFile.close();
    cout << "✅ Item added successfully!\n";
}

void viewItems() {
    ifstream inFile("inventory.dat", ios::binary);
    if (!inFile) {
        cout << "Error opening file!\n";
        return;
    }

    displayHeader("Inventory List");

    Item tempItem;
    cout << left << setw(10) << "ID"
         << setw(25) << "Name"
         << setw(10) << "Qty"
         << setw(10) << "Price" << endl;
    cout << "--------------------------------------------------\n";

    while (inFile.read(reinterpret_cast<char*>(&tempItem), sizeof(Item))) {
        cout << left << setw(10) << tempItem.id
             << setw(25) << tempItem.name
             << setw(10) << tempItem.quantity
             << "$" << fixed << setprecision(2) << tempItem.price << endl;
    }
    inFile.close();
}

void searchItem() {
    ifstream inFile("inventory.dat", ios::binary);
    if (!inFile) {
        cout << "Error opening file!\n";
        return;
    }

    int searchId;
    displayHeader("Search Item");
    cout << "Enter Item ID to search: ";
    cin >> searchId;

    Item tempItem;
    bool found = false;

    while (inFile.read(reinterpret_cast<char*>(&tempItem), sizeof(Item))) {
        if (tempItem.id == searchId) {
            cout << "\n✅ Item Found:\n";
            cout << "ID: " << tempItem.id << endl;
            cout << "Name: " << tempItem.name << endl;
            cout << "Quantity: " << tempItem.quantity << endl;
            cout << "Price: $" << fixed << setprecision(2) << tempItem.price << endl;
            found = true;
            break;
        }
    }

    if (!found) {
        cout << "❌ Item not found!\n";
    }

    inFile.close();
}

void deleteItem() {
    ifstream inFile("inventory.dat", ios::binary);
    if (!inFile) {
        cout << "Error opening file!\n";
        return;
    }

    ofstream tempFile("temp.dat", ios::binary);
    int deleteId;
    displayHeader("Delete Item");
    cout << "Enter Item ID to delete: ";
    cin >> deleteId;

    Item tempItem;
    bool found = false;

    while (inFile.read(reinterpret_cast<char*>(&tempItem), sizeof(Item))) {
        if (tempItem.id != deleteId) {
            tempFile.write(reinterpret_cast<char*>(&tempItem), sizeof(Item));
        } else {
            found = true;
        }
    }

    inFile.close();
    tempFile.close();

    remove("inventory.dat");
    rename("temp.dat", "inventory.dat");

    if (found) {
        cout << "✅ Item deleted successfully!\n";
    } else {
        cout << "❌ Item not found!\n";
    }
}

void updateItem() {
    fstream file("inventory.dat", ios::binary | ios::in | ios::out);
    if (!file) {
        cout << "Error opening file!\n";
        return;
    }

    int updateId;
    displayHeader("Update Item");
    cout << "Enter Item ID to update: ";
    cin >> updateId;

    Item tempItem;
    bool found = false;

    while (file.read(reinterpret_cast<char*>(&tempItem), sizeof(Item))) {
        if (tempItem.id == updateId) {
            cout << "\n✅ Item Found:\n";
            cout << "Current Quantity: " << tempItem.quantity << "\n";
            cout << "Current Price: $" << fixed << setprecision(2) << tempItem.price << "\n";

            cout << "Enter new Quantity: ";
            cin >> tempItem.quantity;
            cout << "Enter new Price: ";
            cin >> tempItem.price;

            file.seekp(-static_cast<long>(sizeof(Item)), ios::cur);
            file.write(reinterpret_cast<char*>(&tempItem), sizeof(Item));
            found = true;
            break;
        }
    }

    if (found) {
        cout << "✅ Item updated successfully!\n";
    } else {
        cout << "❌ Item not found!\n";
    }

    file.close();
}

void displayMenu() {
    int choice;
    do {
        displayHeader("Inventory Management System");
        cout << "1. Add Item\n";
        cout << "2. View Items\n";
        cout << "3. Search Item\n";
        cout << "4. Delete Item\n";
        cout << "5. Update Item\n";
        cout << "6. Exit\n";
        cout << "Enter your choice: ";
        cin >> choice;

        switch (choice) {
            case 1: addItem(); break;
            case 2: viewItems(); break;
            case 3: searchItem(); break;
            case 4: deleteItem(); break;
            case 5: updateItem(); break;
            case 6: cout << "👋 Exiting the system... Goodbye!\n"; break;
            default: cout << "❌ Invalid choice! Please try again.\n"; break;
        }
    } while (choice != 6);
}

int main() {
    displayMenu();
    return 0;
}
