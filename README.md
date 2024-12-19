# HAKATON
#include <iostream> 
#include <fstream> 
#include <vector> 
#include <string> 
#include <algorithm> 
 

using namespace std;

class Expense {
public:
    double amount;
    string category;
    string description;
    //time_t date;

    Expense(double amt, const string& cat, const string& desc)
        : amount(amt), category(cat), description(desc) //date(time(nullptr))
    {}
};

class ExpenseTracker {
private:
    vector<Expense> expenses;

    void merge(vector<Expense>& left, vector<Expense>& right, vector<Expense>& result);
    void mergeSort(vector<Expense>& expenses);

public:
    void addExpense(double amount, const string& category, const string& description);
    void viewExpenses() const;
    double totalExpenses() const;
    void searchByCategory(const string& category) const;
    void saveToFile(const string& filename) const;
    void loadFromFile(const string& filename);
    void sortExpenses(); // метод сортировки расходов
};

void ExpenseTracker::addExpense(double amount, const string& category, const string& description) {
    expenses.emplace_back(amount, category, description);
}

void ExpenseTracker::viewExpenses() const {
    setlocale(LC_ALL, "ru");

    if (expenses.empty()) {
        cout << "Нет записей о расходах." << endl;
        return;
    }

    for (const auto& expense : expenses) {
        cout << "Сумма: " << expense.amount
            << ", Категория: " << expense.category
            << ", Описание: " << expense.description
            << endl;
    }
}

double ExpenseTracker::totalExpenses() const {
    setlocale(LC_ALL, "ru");

    double total = 0.0;
    for (const auto& expense : expenses) {
        total += expense.amount;
    }
    return total;
}

void ExpenseTracker::searchByCategory(const string& category) const {
    setlocale(LC_ALL, "ru");

    bool found = false;
    for (const auto& expense : expenses) {
        if (expense.category == category) {
            found = true;
            std::cout << "Сумма: " << expense.amount
                << ", Описание: " << expense.description
                << endl;
        }
    }

    if (!found) {
        cout << "Расходы по категории \"" << category << "\" не найдены." << endl;
    }
}

void ExpenseTracker::saveToFile(const string& filename) const {
    setlocale(LC_ALL, "ru");

    ofstream file(filename);

    for (const auto& expense : expenses) {
        file << expense.amount << ","
            << expense.category << ","
            << expense.description << ",";
            //<< expense.date << "\n";
    }
}

void ExpenseTracker::loadFromFile(const string& filename) {
    setlocale(LC_ALL, "ru");

    ifstream file(filename);

    if (!file.is_open()) return;

    double amount;
    string category, description;

    while (file >> amount) {
        file.ignore(); // игнорируем запятую 
        getline(file, category, ',');
        getline(file, description, ',');

        expenses.emplace_back(amount, category, description);
        file.ignore(); // игнорируем конец строки 
    }
}

// Merge function for merge sort
void ExpenseTracker::merge(vector<Expense>& left, vector<Expense>& right, vector<Expense>& result) {
    size_t i = 0, j = 0;

    while (i < left.size() && j < right.size()) {
        if (left[i].amount <= right[j].amount) {
            result.push_back(left[i]);
            i++;
        }
        else {
            result.push_back(right[j]);
            j++;
        }
    }

    while (i < left.size()) {
        result.push_back(left[i]);
        i++;
    }

    while (j < right.size()) {
        result.push_back(right[j]);
        j++;
    }
}


void ExpenseTracker::mergeSort(vector<Expense>& expenses) {
    if (expenses.size() <= 1) return;

    size_t mid = expenses.size() / 2;

    vector<Expense> left(expenses.begin(), expenses.begin() + mid);
    vector<Expense> right(expenses.begin() + mid, expenses.end());

    mergeSort(left);
    mergeSort(right);

    expenses.clear();

    merge(left, right, expenses);
}
void ExpenseTracker::sortExpenses() {
    mergeSort(expenses);
}
int main() {
    setlocale(LC_ALL, "ru");
    ExpenseTracker tracker;

    tracker.loadFromFile("expenses.txt");
    cout << "Калькулятор расходов:" << endl;

    while (true) {
        int choice;
        cout << "\n1. Добавить расход\n2. Просмотреть расходы\n3. Общая сумма расходов\n4. Поиск по категории\n5. Сортировать расходы\n6. Выход\n";
        cout << "\nВыберите действие: ";
        cin >> choice;

        if (choice == 1) {
            double amount;
            string category, description;

            cout << "Введите сумму расхода: ";
            while (!(cin >> amount) || amount < 0) {
                cout << "Некорректный ввод. Попробуйте снова: ";
                cin.clear();
                cin.ignore(numeric_limits<streamsize>::max(), '\n');
            }

            cout << "Введите категорию: ";
            cin >> category;
            transform(category.begin(), category.end(), category.begin(),
                ::tolower);

            cout << "Введите описание: ";
            cin.ignore();
            getline(cin, description);

            tracker.addExpense(amount, category, description);

        }
        else if (choice == 2) {
            tracker.viewExpenses();

        }
        else if (choice == 3) {
            double total = tracker.totalExpenses();
            cout << "Общая сумма расходов: " << total << "\n";

        }
        else if (choice == 4) {
            string search_category;
            cout << "Введите категорию для поиска: ";
            cin >> search_category;
            transform(search_category.begin(), search_category.end(), search_category.begin(),
                ::tolower);

            tracker.searchByCategory(search_category);

        }
        else if (choice == 5) { 
            tracker.sortExpenses();
            cout << "Расходы отсортированы.\n";

        }
        else if (choice == 6) { 
            tracker.saveToFile("expenses.txt");
            break;

        }
        else {
            cout << "Некорректный выбор. Попробуйте снова.\n";
        }

        cin.clear();
        while (cin.get() != '\n');
    }

    return 0;
}
