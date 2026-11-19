// Floster.cpp : This file contains the 'main' function. Program execution begins and ends there.
//
// Система обліку (типу "Poster"): товари, каса, витрати/доходи, зарплата
// Visual Studio -> Console App (C++). Кодування файлу: Кирилиця (Windows) - кодова сторінка 1251
#define N0MINMAX
#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <string>
#include <iomanip>
#include <ctime>
#include <limits>
#include <windows.h>
#undef max
#undef min
using namespace std;

// ---------- Структури даних ----------
struct Product {
    int id;
    string name;
    double price;   // ціна продажу
    double cost;    // закупівельна ціна
    int stock;      // залишок на складі
};

struct Employee {
    int id;
    string name;
    string role;
    double rate;    // ставка за годину
    double hours;   // відпрацьовані години (за поточний період)
};

struct Record {
    string date;
    string type;    // "INCOME" або "EXPENSE"
    string desc;
    double amount;
};

vector<Product> products;
vector<Employee> employees;
vector<Record> records;

const string F_PRODUCTS = "products.txt";
const string F_EMPLOYEES = "employees.txt";
const string F_RECORDS = "records.txt";

// ---------- Допоміжні функції ----------
string now() {
    time_t t = time(nullptr);
    tm lt;
    localtime_s(&lt, &t);
    char buf[32];
    strftime(buf, sizeof(buf), "%d.%m.%Y %H:%M", &lt);
    return buf;
}

vector<string> split(const string& s, char d = '|') {
    vector<string> parts;
    string item;
    stringstream ss(s);
    while (getline(ss, item, d)) parts.push_back(item);
    return parts;
}

int readInt(const string& msg) {
    int x;
    while (true) {
        cout << msg;
        if (cin >> x) { cin.ignore(numeric_limits<streamsize>::max(), '\n'); return x; }
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
        cout << "Введіть ціле число!\n";
    }
}

double readDouble(const string& msg) {
    double x;
    while (true) {
        cout << msg;
        if (cin >> x) { cin.ignore(numeric_limits<streamsize>::max(), '\n'); return x; }
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
        cout << "Введіть число!\n";
    }
}

string readLine(const string& msg) {
    string s;
    cout << msg;
    getline(cin, s);
    for (char& c : s) if (c == '|') c = '/';   // '|' - роздільник у файлах
    return s;
}

int nextProductId() { int m = 0; for (auto& p : products) if (p.id > m) m = p.id; return m + 1; }
int nextEmployeeId() { int m = 0; for (auto& e : employees) if (e.id > m) m = e.id; return m + 1; }

Product* findProduct(int id) {
    for (auto& p : products) if (p.id == id) return &p;
    return nullptr;
}
Employee* findEmployee(int id) {
    for (auto& e : employees) if (e.id == id) return &e;
    return nullptr;
}

void addRecord(const string& type, const string& desc, double amount) {
    records.push_back({ now(), type, desc, amount });
}

// ---------- Робота з файлами ----------
void saveAll() {
    ofstream f1(F_PRODUCTS);
    for (auto& p : products)
        f1 << p.id << '|' << p.name << '|' << p.price << '|' << p.cost << '|' << p.stock << '\n';
    ofstream f2(F_EMPLOYEES);
    for (auto& e : employees)
        f2 << e.id << '|' << e.name << '|' << e.role << '|' << e.rate << '|' << e.hours << '\n';
    ofstream f3(F_RECORDS);
    for (auto& r : records)
        f3 << r.date << '|' << r.type << '|' << r.desc << '|' << r.amount << '\n';
}

void loadAll() {
    string line;
    try {
        ifstream f1(F_PRODUCTS);
        while (getline(f1, line)) {
            auto p = split(line);
            if (p.size() == 5) products.push_back({ stoi(p[0]), p[1], stod(p[2]), stod(p[3]), stoi(p[4]) });
        }
        ifstream f2(F_EMPLOYEES);
        while (getline(f2, line)) {
            auto p = split(line);
            if (p.size() == 5) employees.push_back({ stoi(p[0]), p[1], p[2], stod(p[3]), stod(p[4]) });
        }
        ifstream f3(F_RECORDS);
        while (getline(f3, line)) {
            auto p = split(line);
            if (p.size() == 4) records.push_back({ p[0], p[1], p[2], stod(p[3]) });
        }
    }
    catch (...) {
        cout << "Помилка читання файлів даних.\n";
    }
}

// ---------- Товари ----------
void showProducts() {
    cout << "\n" << left << setw(5) << "ID" << setw(25) << "Назва"
        << setw(12) << "Ціна" << setw(12) << "Закупівля" << setw(10) << "Залишок" << "\n";
    cout << string(64, '-') << "\n";
    for (auto& p : products)
        cout << left << setw(5) << p.id << setw(25) << p.name
        << setw(12) << fixed << setprecision(2) << p.price
        << setw(12) << p.cost << setw(10) << p.stock << "\n";
    if (products.empty()) cout << "(товарів ще немає)\n";
}

void addProduct() {
    Product p;
    p.id = nextProductId();
    p.name = readLine("Назва товару: ");
    p.price = readDouble("Ціна продажу: ");
    p.cost = readDouble("Закупівельна ціна: ");
    p.stock = readInt("Кількість на складі: ");
    products.push_back(p);
    cout << "Товар додано.\n";
}

void editProduct() {
    showProducts();
    Product* p = findProduct(readInt("ID товару для редагування: "));
    if (!p) { cout << "Не знайдено.\n"; return; }
    p->name = readLine("Нова назва: ");
    p->price = readDouble("Нова ціна продажу: ");
    p->cost = readDouble("Нова закупівельна ціна: ");
    cout << "Збережено.\n";
}

void deleteProduct() {
    showProducts();
    int id = readInt("ID товару для видалення: ");
    for (size_t i = 0; i < products.size(); i++)
        if (products[i].id == id) { products.erase(products.begin() + i); cout << "Видалено.\n"; return; }
    cout << "Не знайдено.\n";
}

void restock() {
    showProducts();
    Product* p = findProduct(readInt("ID товару для поповнення: "));
    if (!p) { cout << "Не знайдено.\n"; return; }
    int qty = readInt("Скільки одиниць закуплено: ");
    if (qty <= 0) return;
    p->stock += qty;
    double sum = qty * p->cost;
    addRecord("EXPENSE", "Закупівля: " + p->name + " x" + to_string(qty), sum);
    cout << "Склад поповнено. Витрата: " << sum << " грн\n";
}

// ---------- Каса (продаж) ----------
void sell() {
    struct Item { int id; int qty; };
    vector<Item> cart;
    double total = 0;
    while (true) {
        showProducts();
        int id = readInt("ID товару (0 - завершити чек): ");
        if (id == 0) break;
        Product* p = findProduct(id);
        if (!p) { cout << "Не знайдено.\n"; continue; }
        int qty = readInt("Кількість: ");
        int already = 0;
        for (auto& it : cart) if (it.id == id) already += it.qty;
        if (qty <= 0 || qty + already > p->stock) { cout << "Недостатньо на складі (є " << p->stock - already << ").\n"; continue; }
        cart.push_back({ id, qty });
        total += qty * p->price;
        cout << "Додано. Сума чека: " << fixed << setprecision(2) << total << " грн\n";
    }
    if (cart.empty()) return;

    cout << "\n===== ЧЕК =====\n";
    for (auto& it : cart) {
        Product* p = findProduct(it.id);
        cout << p->name << "  " << it.qty << " x " << p->price << " = " << it.qty * p->price << "\n";
    }
    cout << "РАЗОМ: " << total << " грн\n";

    int ok = readInt("Підтвердити оплату? (1 - так, 0 - скасувати): ");
    if (ok != 1) { cout << "Скасовано.\n"; return; }

    double given = readDouble("Отримано від клієнта: ");
    if (given < total) { cout << "Недостатньо коштів. Чек скасовано.\n"; return; }
    cout << "Решта: " << given - total << " грн\n";

    for (auto& it : cart) findProduct(it.id)->stock -= it.qty;
    addRecord("INCOME", "Продаж (чек, позицій: " + to_string(cart.size()) + ")", total);
    cout << "Продаж проведено.\n";
}

// ---------- Витрати і доходи ----------
void addManualRecord(const string& type) {
    string desc = readLine("Опис: ");
    double amount = readDouble("Сума: ");
    addRecord(type, desc, amount);
    cout << "Записано.\n";
}

void showRecords() {
    cout << "\n" << left << setw(18) << "Дата" << setw(10) << "Тип" << setw(40) << "Опис" << "Сума\n";
    cout << string(76, '-') << "\n";
    for (auto& r : records)
        cout << left << setw(18) << r.date << setw(10) << (r.type == "INCOME" ? "Дохід" : "Витрата")
        << setw(40) << r.desc << fixed << setprecision(2) << r.amount << "\n";
    if (records.empty()) cout << "(записів немає)\n";
}

void report() {
    double inc = 0, exp = 0;
    for (auto& r : records) (r.type == "INCOME" ? inc : exp) += r.amount;
    cout << fixed << setprecision(2);
    cout << "\n===== ЗВІТ =====\n";
    cout << "Доходи:  " << inc << " грн\n";
    cout << "Витрати: " << exp << " грн\n";
    cout << "Прибуток: " << inc - exp << " грн\n";
    double stockValue = 0;
    for (auto& p : products) stockValue += p.stock * p.cost;
    cout << "Вартість залишків на складі (за закупівлею): " << stockValue << " грн\n";
}

// ---------- Працівники і зарплата ----------
void showEmployees() {
    cout << "\n" << left << setw(5) << "ID" << setw(25) << "ПІБ" << setw(15) << "Посада"
        << setw(12) << "Ставка/год" << setw(10) << "Годин" << setw(12) << "До виплати" << "\n";
    cout << string(79, '-') << "\n";
    for (auto& e : employees)
        cout << left << setw(5) << e.id << setw(25) << e.name << setw(15) << e.role
        << setw(12) << fixed << setprecision(2) << e.rate << setw(10) << e.hours
        << setw(12) << e.rate * e.hours << "\n";
    if (employees.empty()) cout << "(працівників немає)\n";
}

void addEmployee() {
    Employee e;
    e.id = nextEmployeeId();
    e.name = readLine("ПІБ: ");
    e.role = readLine("Посада: ");
    e.rate = readDouble("Ставка за годину (грн): ");
    e.hours = 0;
    employees.push_back(e);
    cout << "Працівника додано.\n";
}

void addHours() {
    showEmployees();
    Employee* e = findEmployee(readInt("ID працівника: "));
    if (!e) { cout << "Не знайдено.\n"; return; }
    double h = readDouble("Скільки годин додати: ");
    if (h > 0) e->hours += h;
    cout << "Години додано.\n";
}

void paySalary() {
    showEmployees();
    Employee* e = findEmployee(readInt("ID працівника для виплати: "));
    if (!e) { cout << "Не знайдено.\n"; return; }
    double bonus = readDouble("Премія (0 якщо немає): ");
    double sum = e->rate * e->hours + bonus;
    if (sum <= 0) { cout << "Нічого виплачувати.\n"; return; }
    cout << "До виплати: " << fixed << setprecision(2) << sum << " грн\n";
    if (readInt("Підтвердити (1 - так, 0 - ні): ") != 1) return;
    addRecord("EXPENSE", "Зарплата: " + e->name, sum);
    e->hours = 0;
    cout << "Виплачено, години обнулено.\n";
}

// ---------- Меню ----------
void productsMenu() {
    while (true) {
        cout << "\n--- ТОВАРИ ---\n1. Показати\n2. Додати\n3. Редагувати\n4. Видалити\n5. Поповнити склад\n0. Назад\n";
        int c = readInt("Вибір: ");
        if (c == 0) return;
        if (c == 1) showProducts();
        else if (c == 2) addProduct();
        else if (c == 3) editProduct();
        else if (c == 4) deleteProduct();
        else if (c == 5) restock();
        saveAll();
    }
}

void financeMenu() {
    while (true) {
        cout << "\n--- ФІНАНСИ ---\n1. Додати витрату\n2. Додати дохід\n3. Журнал операцій\n4. Звіт\n0. Назад\n";
        int c = readInt("Вибір: ");
        if (c == 0) return;
        if (c == 1) addManualRecord("EXPENSE");
        else if (c == 2) addManualRecord("INCOME");
        else if (c == 3) showRecords();
        else if (c == 4) report();
        saveAll();
    }
}

void staffMenu() {
    while (true) {
        cout << "\n--- ПРАЦІВНИКИ ---\n1. Список\n2. Додати працівника\n3. Додати відпрацьовані години\n4. Виплатити зарплату\n0. Назад\n";
        int c = readInt("Вибір: ");
        if (c == 0) return;
        if (c == 1) showEmployees();
        else if (c == 2) addEmployee();
        else if (c == 3) addHours();
        else if (c == 4) paySalary();
        saveAll();
    }
}

int main() {
    SetConsoleCP(CP_UTF8);
    SetConsoleOutputCP(CP_UTF8);
    loadAll();

    while (true) {
        cout << "\n====== ГОЛОВНЕ МЕНЮ ======\n1. Каса (продаж)\n2. Товари\n3. Фінанси\n4. Працівники і зарплата\n0. Вихід\n";
        int c = readInt("Вибір: ");
        if (c == 0) break;
        if (c == 1) sell();
        else if (c == 2) productsMenu();
        else if (c == 3) financeMenu();
        else if (c == 4) staffMenu();
        saveAll();
    }
    saveAll();
    cout << "До побачення!\n";
    return 0;
}
