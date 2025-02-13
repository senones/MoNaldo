# MoNaldo
MyFirst IOS APP


import SwiftUI

struct ExpenseEntry: Identifiable, Codable {
    let id = UUID()
    let amount: Double
    let name: String
    let category: String
    let date: Date
}

struct IncomeEntry: Identifiable, Codable {
    let id = UUID()
    let amount: Double
    let name: String
    let category: String
    let date: Date
}

struct MoNaldoColors {
    // Hauptfarben
    static let darkBrown = Color(hex: "1C1C1E") // Dunkler für Dark Mode
    static let gold = Color(hex: "FFD700")      // Kräftigeres Gold
    static let mint = Color(hex: "4CD964")      // Frischeres Mint
    
    // Adaptive Farben
    static var adaptiveText: Color {
        Color(.label)
    }
    static var adaptiveSecondary: Color {
        Color(.secondaryLabel)
    }
    static var adaptiveBackground: Color {
        Color(.systemBackground)
    }
    static var adaptiveGroupedBackground: Color {
        Color(.systemGroupedBackground)
    }
    static var adaptiveSecondaryBackground: Color {
        Color(.secondarySystemBackground)
    }
}

extension Color {
    init(hex: String) {
        let hex = hex.trimmingCharacters(in: CharacterSet.alphanumerics.inverted)
        var int: UInt64 = 0
        Scanner(string: hex).scanHexInt64(&int)
        let a, r, g, b: UInt64
        switch hex.count {
        case 3: // RGB (12-bit)
            (a, r, g, b) = (255, (int >> 8) * 17, (int >> 4 & 0xF) * 17, (int & 0xF) * 17)
        case 6: // RGB (24-bit)
            (a, r, g, b) = (255, int >> 16, int >> 8 & 0xFF, int & 0xFF)
        case 8: // ARGB (32-bit)
            (a, r, g, b) = (int >> 24, int >> 16 & 0xFF, int >> 8 & 0xFF, int & 0xFF)
        default:
            (a, r, g, b) = (1, 1, 1, 0)
        }
        self.init(
            .sRGB,
            red: Double(r) / 255,
            green: Double(g) / 255,
            blue:  Double(b) / 255,
            opacity: Double(a) / 255
        )
    }
}

struct CustomButtonStyle: ButtonStyle {
    let backgroundColor: Color
    
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding(.horizontal, 20)
            .padding(.vertical, 12)
            .background(MoNaldoColors.darkBrown)
            .foregroundColor(MoNaldoColors.gold)
            .clipShape(RoundedRectangle(cornerRadius: 12))
            .shadow(color: Color.black.opacity(0.1), radius: 4, x: 0, y: 2)
            .scaleEffect(configuration.isPressed ? 0.98 : 1)
            .animation(.easeInOut(duration: 0.2), value: configuration.isPressed)
    }
}

enum Utilities {
    static func getCategoryIcon(_ category: String) -> String {
        switch category {
        case "Lebensmittel":
            return "cart.fill"
        case "Transport":
            return "car.fill"
        case "Miete":
            return "house.fill"
        case "Unterhaltung":
            return "tv.fill"
        case "Shopping":
            return "bag.fill"
        case "Gehalt":
            return "dollarsign.circle.fill"
        case "Nebenjob":
            return "briefcase.fill"
        case "Geschenk":
            return "gift.fill"
        case "Verkauf":
            return "tag.fill"
        default:
            return "creditcard.fill"
        }
    }
}

struct ExpensesView: View {
    let expenseEntries: [ExpenseEntry]
    let onDelete: (ExpenseEntry) -> Void
    let currency: Currency
    
    var body: some View {
        NavigationView {
            VStack {
                // Summary Cards
                HStack(spacing: 15) {
                    SummaryCard(
                        title: "Gesamt Ausgaben",
                        amount: expenseEntries.reduce(0) { $0 + $1.amount },
                        color: .red,
                        currency: currency
                    )
                    
                    SummaryCard(
                        title: "Durchschnitt",
                        amount: expenseEntries.isEmpty ? 0 : expenseEntries.reduce(0) { $0 + $1.amount } / Double(expenseEntries.count),
                        color: MoNaldoColors.adaptiveText,
                        currency: currency
                    )
                }
                .padding()
                
                // Expenses List
                List {
                    ForEach(groupExpensesByDate(), id: \.date) { group in
                        Section(header: Text(formatDateHeader(group.date))) {
                            ForEach(group.expenses) { entry in
                                HStack(alignment: .center) {
                                    Image(systemName: Utilities.getCategoryIcon(entry.category))
                                        .foregroundColor(.red)
                                        .frame(width: 30)
                                    
                                    VStack(alignment: .leading) {
                                        Text(entry.name)
                                            .font(.headline)
                                            .foregroundColor(MoNaldoColors.adaptiveText)
                                        Text(entry.category)
                                            .font(.caption)
                                            .foregroundColor(MoNaldoColors.adaptiveSecondary)
                                    }
                                    
                                    Spacer()
                                    
                                    VStack(alignment: .trailing) {
                                        Text("-\(entry.amount, specifier: "%.2f") \(currency.symbol)")
                                            .font(.title2)
                                            .bold()
                                            .foregroundColor(.red)
                                            .shadow(color: .red.opacity(0.3), radius: 2, x: 0, y: 0)
                                    }
                                }
                                .background(Color(.systemBackground))
                                .swipeActions(edge: .trailing) {
                                    Button(role: .destructive) {
                                        onDelete(entry)
                                    } label: {
                                        Label("Löschen", systemImage: "trash")
                                    }
                                }
                            }
                        }
                    }
                }
                .listStyle(InsetGroupedListStyle())
                .background(MoNaldoColors.adaptiveGroupedBackground)
            }
            .navigationTitle("MoNaldo")
            .navigationBarTitleDisplayMode(.inline)
            .toolbarColorScheme(.dark, for: .navigationBar)
            .toolbarBackground(MoNaldoColors.darkBrown, for: .navigationBar)
            .toolbarBackground(.visible, for: .navigationBar)
        }
    }
    
    private struct ExpenseGroup {
        let date: Date
        let expenses: [ExpenseEntry]
    }
    
    private func groupExpensesByDate() -> [ExpenseGroup] {
        let calendar = Calendar.current
        let grouped = Dictionary(grouping: expenseEntries) { entry in
            calendar.startOfDay(for: entry.date)
        }
        
        return grouped.map { ExpenseGroup(date: $0.key, expenses: $0.value.sorted(by: { $0.date > $1.date })) }
            .sorted(by: { $0.date > $1.date })
    }
    
    private func formatDateHeader(_ date: Date) -> String {
        let formatter = DateFormatter()
        formatter.dateStyle = .full
        return formatter.string(from: date)
    }
    
    private func formatTime(_ date: Date) -> String {
        let formatter = DateFormatter()
        formatter.timeStyle = .short
        return formatter.string(from: date)
    }
}

struct SummaryCard: View {
    let title: String
    let amount: Double
    let color: Color
    let currency: Currency
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(title)
                .font(.subheadline)
                .foregroundColor(MoNaldoColors.adaptiveText)
            Text("\(amount, specifier: "%.2f") \(currency.symbol)")
                .font(.title2.bold())
                .foregroundColor(color)
        }
        .frame(maxWidth: .infinity, alignment: .leading)
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 12)
                .fill(MoNaldoColors.adaptiveSecondaryBackground)
                .shadow(
                    color: Color.black.opacity(0.05),
                    radius: 10,
                    x: 0,
                    y: 5
                )
        )
    }
}

struct IncomeView: View {
    let incomeEntries: [IncomeEntry]
    let onDelete: (IncomeEntry) -> Void
    let currency: Currency
    
    var body: some View {
        NavigationView {
            List {
                if incomeEntries.isEmpty {
                    Text("Keine Einnahmen vorhanden")
                        .foregroundColor(MoNaldoColors.adaptiveText)
                } else {
                    ForEach(incomeEntries.sorted(by: { $0.date > $1.date })) { income in
                        HStack {
                            Image(systemName: Utilities.getCategoryIcon(income.category))
                                .foregroundColor(.green)
                                .font(.title2)
                            
                            VStack(alignment: .leading) {
                                Text(income.name)
                                    .font(.headline)
                                Text(income.category)
                                    .font(.subheadline)
                                    .foregroundColor(MoNaldoColors.adaptiveText)
                            }
                            
                            Spacer()
                            
                            Text("+\(income.amount, specifier: "%.2f") \(currency.symbol)")
                                .font(.title2)
                                .bold()
                                .foregroundColor(.green)
                                .shadow(color: Color.green.opacity(0.3), radius: 2, x: 0, y: 0)
                        }
                        .swipeActions(edge: .trailing) {
                            Button(role: .destructive) {
                                onDelete(income)
                            } label: {
                                Label("Löschen", systemImage: "trash")
                            }
                        }
                    }
                }
            }
            .navigationTitle("MoNaldo")
            .navigationBarTitleDisplayMode(.inline)
            .toolbarColorScheme(.dark, for: .navigationBar)
            .toolbarBackground(MoNaldoColors.darkBrown, for: .navigationBar)
            .toolbarBackground(.visible, for: .navigationBar)
        }
    }
}

// Update Currency enum to keep only EUR, USD and CHF
enum Currency: String, CaseIterable {
    case eur = "€"
    case usd = "$"
    case chf = "CHF"
    
    var symbol: String {
        return self.rawValue
    }
    
    var icon: String {
        switch self {
        case .eur:
            return "eurosign.circle.fill"
        case .usd:
            return "dollarsign.circle.fill"
        case .chf:
            return "francsign.circle.fill"
        }
    }
    
    var name: String {
        switch self {
        case .eur:
            return "Euro"
        case .usd:
            return "US-Dollar"
        case .chf:
            return "Schweizer Franken"
        }
    }
}

// Create a new struct for the Balance Card
struct BalanceCardView: View {
    let balance: Double
    let selectedCurrency: Currency
    let showCurrencyPicker: () -> Void
    
    var body: some View {
        VStack(spacing: 10) {
            Text("Verfügbares Geld")
                .font(.headline)
                .foregroundColor(MoNaldoColors.adaptiveText)
            HStack {
                Text("\(balance, specifier: "%.2f")")
                    .font(.system(size: 34, weight: .bold))
                    .foregroundColor(balance >= 0 ? .green : .red)
                
                Button(action: showCurrencyPicker) {
                    Text(selectedCurrency.symbol)
                        .font(.system(size: 34, weight: .bold))
                        .foregroundColor(balance >= 0 ? .green : .red)
                }
            }
        }
        .frame(maxWidth: .infinity)
        .padding(.vertical, 20)
        .background(
            RoundedRectangle(cornerRadius: 15)
                .fill(MoNaldoColors.adaptiveSecondaryBackground)
                .shadow(
                    color: Color.black.opacity(0.05),
                    radius: 10,
                    x: 0,
                    y: 5
                )
        )
        .padding(.horizontal)
    }
}

// Update InputFieldView to handle keyboard
struct InputFieldView: View {
    let title: String
    let placeholder: String
    @Binding var text: String
    var keyboardType: UIKeyboardType = .default
    @FocusState private var isFocused: Bool
    
    var body: some View {
        TextField(placeholder, text: $text)
            .textFieldStyle(RoundedBorderTextFieldStyle())
            .keyboardType(keyboardType)
            .focused($isFocused)
            .padding()
            .background(
                RoundedRectangle(cornerRadius: 10)
                    .fill(Color(.systemBackground))
                    .overlay(
                        RoundedRectangle(cornerRadius: 10)
                            .stroke(MoNaldoColors.mint.opacity(0.3), lineWidth: 1)
                    )
            )
            .submitLabel(.done)
            .onSubmit {
                isFocused = false
            }
    }
}

// Update TransactionInputView to handle keyboard better
struct TransactionInputView: View {
    let title: String
    @Binding var name: String
    @Binding var amount: String
    let buttonAction: () -> Void
    let buttonDisabled: Bool
    @FocusState private var nameFocused: Bool
    @FocusState private var amountFocused: Bool
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(title)
                .font(.title3)
                .bold()
                .foregroundColor(MoNaldoColors.adaptiveText)
            
            TextField("Name der \(title)", text: $name)
                .textFieldStyle(RoundedBorderTextFieldStyle())
                .focused($nameFocused)
                .submitLabel(.next)
                .foregroundColor(MoNaldoColors.adaptiveText)
                .background(
                    RoundedRectangle(cornerRadius: 10)
                        .fill(MoNaldoColors.adaptiveBackground)
                )
            
            HStack {
                TextField("Betrag eingeben", text: $amount)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .keyboardType(.decimalPad)
                    .focused($amountFocused)
                    .submitLabel(.done)
                    .foregroundColor(MoNaldoColors.adaptiveText)
                    .background(
                        RoundedRectangle(cornerRadius: 10)
                            .fill(MoNaldoColors.adaptiveBackground)
                    )
                
                Button("Kategorie") {
                    nameFocused = false
                    amountFocused = false
                    buttonAction()
                }
                .buttonStyle(CustomButtonStyle(backgroundColor: MoNaldoColors.darkBrown))
                .disabled(buttonDisabled)
            }
        }
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 15)
                .fill(MoNaldoColors.adaptiveBackground)
                .overlay(
                    RoundedRectangle(cornerRadius: 15)
                        .stroke(MoNaldoColors.mint.opacity(0.2), lineWidth: 1)
                )
        )
        .padding(.horizontal)
    }
}

// Update TransactionListView to remove GeometryReader
struct TransactionListView: View {
    let incomeEntries: [IncomeEntry]
    let expenseEntries: [ExpenseEntry]
    let selectedCurrency: Currency
    let onDeleteIncome: (Int) -> Void
    let onDeleteExpense: (ExpenseEntry) -> Void
    
    var body: some View {
        VStack(spacing: 0) {
            // Income Section
            Section {
                if incomeEntries.isEmpty {
                    Text("Keine Einnahmen vorhanden")
                        .foregroundColor(MoNaldoColors.adaptiveText)
                        .padding()
                } else {
                    ForEach(incomeEntries.indices, id: \.self) { index in
                        IncomeRowView(
                            income: incomeEntries[index],
                            currency: selectedCurrency,
                            onDelete: { onDeleteIncome(index) }
                        )
                    }
                }
            } header: {
                SectionHeaderView(title: "Einnahmen")
            }
            
            // Expenses Section
            Section {
                if expenseEntries.isEmpty {
                    EmptyStateView()
                } else {
                    ForEach(expenseEntries.sorted(by: { $0.date > $1.date })) { entry in
                        ExpenseRowView(
                            entry: entry,
                            currency: selectedCurrency,
                            onDelete: { onDeleteExpense(entry) }
                        )
                    }
                }
            } header: {
                SectionHeaderView(title: "Ausgaben")
            }
        }
        .padding(.bottom, 10)
    }
}

// 2. Erstelle separate Views für die Einträge
struct IncomeRowView: View {
    let income: IncomeEntry
    let currency: Currency
    let onDelete: () -> Void
    
    var body: some View {
        HStack {
            Image(systemName: Utilities.getCategoryIcon(income.category))
                .foregroundColor(.green)
                .font(.title2)
            
            VStack(alignment: .leading) {
                Text(income.name)
                    .font(.headline)
                    .foregroundColor(MoNaldoColors.adaptiveText)
                Text(income.category)
                    .font(.subheadline)
                    .foregroundColor(MoNaldoColors.adaptiveSecondary)
            }
            
            Spacer()
            
            VStack(alignment: .trailing) {
                Text("+\(income.amount, specifier: "%.2f") \(currency.symbol)")
                    .font(.title2)
                    .bold()
                    .foregroundColor(.green)
                    .shadow(color: Color.green.opacity(0.3), radius: 2, x: 0, y: 0)
                
                Button(action: onDelete) {
                    Image(systemName: "trash")
                        .foregroundColor(.red)
                }
            }
        }
        .swipeActions(edge: .trailing) {
            Button(role: .destructive) {
                onDelete()
            } label: {
                Label("Löschen", systemImage: "trash")
            }
        }
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 10)
                .fill(MoNaldoColors.adaptiveBackground)
                .overlay(
                    RoundedRectangle(cornerRadius: 10)
                        .stroke(MoNaldoColors.darkBrown.opacity(0.2), lineWidth: 1)
                )
        )
        .padding(.horizontal)
    }
}

// 1. SectionHeaderView
struct SectionHeaderView: View {
    let title: String
    
    var body: some View {
        Text(title)
            .font(.headline)
            .foregroundColor(MoNaldoColors.adaptiveText)
            .frame(maxWidth: .infinity, alignment: .leading)
            .padding()
            .background(MoNaldoColors.gold.opacity(0.15))
    }
}

// 2. EmptyStateView
struct EmptyStateView: View {
    var body: some View {
        Text("Keine Ausgaben vorhanden")
            .foregroundColor(MoNaldoColors.adaptiveText)
            .padding()
    }
}

// 3. ExpenseRowView
struct ExpenseRowView: View {
    let entry: ExpenseEntry
    let currency: Currency
    let onDelete: () -> Void
    
    var body: some View {
        HStack {
            Image(systemName: Utilities.getCategoryIcon(entry.category))
                .foregroundColor(.red)
                .font(.title2)
                .frame(width: 30)
            
            VStack(alignment: .leading) {
                Text(entry.name)
                    .font(.headline)
                    .foregroundColor(MoNaldoColors.adaptiveText)
                Text(entry.category)
                    .font(.subheadline)
                    .foregroundColor(MoNaldoColors.adaptiveSecondary)
                Text(formatDate(entry.date))
                    .font(.caption)
                    .foregroundColor(MoNaldoColors.adaptiveSecondary)
            }
            
            Spacer()
            
            VStack(alignment: .trailing) {
                Text("-\(entry.amount, specifier: "%.2f") \(currency.symbol)")
                    .font(.title2)
                    .bold()
                    .foregroundColor(.red)
                    .shadow(color: .red.opacity(0.3), radius: 2, x: 0, y: 0)
                
                Button(action: onDelete) {
                    Image(systemName: "trash")
                        .foregroundColor(.red)
                }
            }
        }
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 10)
                .fill(MoNaldoColors.adaptiveBackground)
                .overlay(
                    RoundedRectangle(cornerRadius: 10)
                        .stroke(MoNaldoColors.darkBrown.opacity(0.2), lineWidth: 1)
                )
        )
        .padding(.horizontal)
    }
    
    private func formatDate(_ date: Date) -> String {
        let formatter = DateFormatter()
        formatter.dateStyle = .medium
        formatter.timeStyle = .short
        return formatter.string(from: date)
    }
}

// Update PieChartView to fix the chart visibility
struct PieChartView: View {
    let expenses: [ExpenseEntry]
    let incomes: [IncomeEntry]
    let balance: Double
    let currency: Currency
    
    var body: some View {
        VStack {
            // Monthly Summary Title
            Text("Monatsübersicht")
                .font(.headline)
                .foregroundColor(MoNaldoColors.adaptiveText)
                .padding(.top)
            
            // Pie Chart
            ZStack {
                Circle()
                    .trim(from: 0, to: expensesRatio)
                    .stroke(Color.red, lineWidth: 30)
                    .rotationEffect(.degrees(-90))
                
                Circle()
                    .trim(from: expensesRatio, to: expensesRatio + incomesRatio)
                    .stroke(MoNaldoColors.mint, lineWidth: 30)
                    .rotationEffect(.degrees(-90))
                
                Circle()
                    .trim(from: expensesRatio + incomesRatio, to: 1)
                    .stroke(MoNaldoColors.gold, lineWidth: 30)
                    .rotationEffect(.degrees(-90))
            }
            .frame(width: 200, height: 200) // Feste Größe statt GeometryReader
            .padding()
            
            // Legend
            VStack(alignment: .leading, spacing: 8) {
                LegendItem(
                    color: .red,
                    text: String(format: "Ausgaben: %.2f %@", totalExpenses, currency.symbol),
                    textColor: MoNaldoColors.adaptiveText
                )
                LegendItem(
                    color: MoNaldoColors.mint,
                    text: String(format: "Einnahmen: %.2f %@", totalIncomes, currency.symbol),
                    textColor: MoNaldoColors.adaptiveText
                )
                LegendItem(
                    color: MoNaldoColors.gold,
                    text: String(format: "Verfügbar: %.2f %@", balance, currency.symbol),
                    textColor: MoNaldoColors.adaptiveText
                )
            }
            .padding()
        }
        .background(
            RoundedRectangle(cornerRadius: 15)
                .fill(MoNaldoColors.adaptiveSecondaryBackground)
                .shadow(
                    color: Color.black.opacity(0.05),
                    radius: 10,
                    x: 0,
                    y: 5
                )
        )
        .padding(.horizontal)
    }
    
    private var totalExpenses: Double {
        expenses.reduce(0) { $0 + $1.amount }
    }
    
    private var totalIncomes: Double {
        incomes.reduce(0) { $0 + $1.amount }
    }
    
    private var total: Double {
        totalExpenses + totalIncomes + balance
    }
    
    private var expensesRatio: Double {
        total == 0 ? 0 : Double(totalExpenses) / total
    }
    
    private var incomesRatio: Double {
        total == 0 ? 0 : Double(totalIncomes) / total
    }
}

// Update LegendItem to accept textColor parameter
struct LegendItem: View {
    let color: Color
    let text: String
    let textColor: Color
    
    var body: some View {
        HStack {
            Circle()
                .fill(color)
                .frame(width: 20, height: 20)
            Text(text)
                .foregroundColor(textColor)
        }
    }
}

// Update ContentView to include PieChartView
struct ContentView: View {
    @State private var salary: String = ""
    @State private var income: String = ""
    @State private var expense: String = ""
    @State private var expenseName: String = ""
    @State private var balance: Double = 0.0
    @State private var showingCategorySheet = false
    
    // Lists to store entries
    @State private var incomeEntries: [IncomeEntry] = []
    @State private var expenseEntries: [ExpenseEntry] = []
    
    // Predefined expense categories
    @State private var categories: [String] = ["Lebensmittel", "Transport", "Miete", "Unterhaltung", "Shopping", "Sonstiges"]
    @State private var newCategory: String = ""
    @State private var showingAddCategorySheet = false
    @State private var showingSalaryAlert = false
    
    // Keys for UserDefaults
    private let balanceKey = "savedBalance"
    private let incomeEntriesKey = "savedIncomeEntries"
    private let expenseEntriesKey = "savedExpenseEntries"
    
    // Add this new property to track the current monthly salary
    @State private var currentMonthlySalary: Double = 0.0
    
    @State private var incomeName: String = ""
    @State private var incomeCategory: String = ""
    @State private var showingIncomeCategorySheet = false
    
    // Add income categories
    @State private var incomeCategories: [String] = ["Gehalt", "Nebenjob", "Geschenk", "Verkauf", "Sonstiges"]
    
    // Add new UserDefaults key
    private let incomeCategoriesKey = "savedIncomeCategories"
    
    @State private var selectedCurrency: Currency = .eur
    @State private var showingCurrencyPicker = false
    
    // Add this to your existing properties
    private let currencyKey = "savedCurrency"
    
    @State private var showingAddIncomeCategorySheet = false
    @State private var newIncomeCategory: String = ""
    
    // Add minimum balance properties to ContentView
    @State private var minimumBalance: Double = 0.0
    @State private var showingMinimumBalanceAlert = false
    private let minimumBalanceKey = "minimumBalance"
    
    var body: some View {
        TabView {
            NavigationView {
                ZStack {
                    MoNaldoColors.adaptiveBackground
                        .ignoresSafeArea()
                    
                    ScrollView(.vertical, showsIndicators: true) {
                        VStack(spacing: 20) {
                            // Top Section with Balance and PieChart
                            if UIScreen.main.bounds.width > 700 { // iPad Layout
                                HStack(alignment: .top, spacing: 20) {
                                    BalanceCardView(
                                        balance: balance,
                                        selectedCurrency: selectedCurrency,
                                        showCurrencyPicker: { showingCurrencyPicker = true }
                                    )
                                    .frame(maxWidth: UIScreen.main.bounds.width * 0.45)
                                    
                                    PieChartView(
                                        expenses: expenseEntries,
                                        incomes: incomeEntries,
                                        balance: balance,
                                        currency: selectedCurrency
                                    )
                                    .frame(maxWidth: UIScreen.main.bounds.width * 0.45)
                                }
                                .padding(.horizontal)
                                
                                // Input Section
                                HStack(spacing: 20) {
                                    VStack {
                                        MinimumBalanceInputView(
                                            minimumBalance: $minimumBalance,
                                            currency: selectedCurrency
                                        )
                                        
                                        SalaryInputView(
                                            salary: $salary,
                                            currentMonthlySalary: $currentMonthlySalary,
                                            showingSalaryAlert: $showingSalaryAlert
                                        )
                                    }
                                    .frame(maxWidth: UIScreen.main.bounds.width * 0.45)
                                    
                                    VStack {
                                        TransactionInputView(
                                            title: "Einnahme",
                                            name: $incomeName,
                                            amount: $income,
                                            buttonAction: { showingIncomeCategorySheet = true },
                                            buttonDisabled: incomeName.isEmpty
                                        )
                                        
                                        TransactionInputView(
                                            title: "Ausgabe",
                                            name: $expenseName,
                                            amount: $expense,
                                            buttonAction: { showingCategorySheet = true },
                                            buttonDisabled: expenseName.isEmpty
                                        )
                                    }
                                    .frame(maxWidth: UIScreen.main.bounds.width * 0.45)
                                }
                                .padding(.horizontal)
                            } else { // iPhone Layout
                                BalanceCardView(
                                    balance: balance,
                                    selectedCurrency: selectedCurrency,
                                    showCurrencyPicker: { showingCurrencyPicker = true }
                                )
                                
                                PieChartView(
                                    expenses: expenseEntries,
                                    incomes: incomeEntries,
                                    balance: balance,
                                    currency: selectedCurrency
                                )
                                
                                MinimumBalanceInputView(
                                    minimumBalance: $minimumBalance,
                                    currency: selectedCurrency
                                )
                                
                                SalaryInputView(
                                    salary: $salary,
                                    currentMonthlySalary: $currentMonthlySalary,
                                    showingSalaryAlert: $showingSalaryAlert
                                )
                                
                                TransactionInputView(
                                    title: "Einnahme",
                                    name: $incomeName,
                                    amount: $income,
                                    buttonAction: { showingIncomeCategorySheet = true },
                                    buttonDisabled: incomeName.isEmpty
                                )
                                
                                TransactionInputView(
                                    title: "Ausgabe",
                                    name: $expenseName,
                                    amount: $expense,
                                    buttonAction: { showingCategorySheet = true },
                                    buttonDisabled: expenseName.isEmpty
                                )
                            }
                            
                            // Transaction List
                            TransactionListView(
                                incomeEntries: incomeEntries,
                                expenseEntries: expenseEntries,
                                selectedCurrency: selectedCurrency,
                                onDeleteIncome: deleteIncome,
                                onDeleteExpense: deleteExpense
                            )
                            .frame(maxWidth: UIScreen.main.bounds.width > 700 ? UIScreen.main.bounds.width * 0.9 : .infinity)
                        }
                    }
                    .padding(.vertical)
                    .padding(.bottom, 20)
                }
                .frame(maxWidth: .infinity)
                .background(MoNaldoColors.adaptiveBackground)
                .ignoresSafeArea(.keyboard)
                .navigationTitle("MoNaldo")
                .navigationBarTitleDisplayMode(.inline)
                .toolbarColorScheme(.dark, for: .navigationBar)
                .toolbarBackground(MoNaldoColors.darkBrown, for: .navigationBar)
                .toolbarBackground(.visible, for: .navigationBar)
                .sheet(isPresented: $showingCategorySheet) {
                    NavigationView {
                        List {
                            ForEach(categories, id: \.self) { category in
                                Button(action: {
                                    addExpense(category: category)
                                    showingCategorySheet = false
                                }) {
                                    Text(category)
                                        .foregroundColor(MoNaldoColors.adaptiveText)
                                }
                            }
                            .onDelete(perform: deleteCategory)
                            
                            Button(action: {
                                showingCategorySheet = false
                                showingAddCategorySheet = true
                            }) {
                                HStack {
                                    Image(systemName: "plus.circle.fill")
                                    Text("Neue Kategorie")
                                }
                                .foregroundColor(MoNaldoColors.adaptiveText)
                            }
                        }
                        .navigationTitle("Kategorie wählen")
                        .navigationBarItems(
                            trailing: Button("Abbrechen") {
                                showingCategorySheet = false
                            }
                        )
                    }
                }
                .sheet(isPresented: $showingAddCategorySheet) {
                    NavigationView {
                        Form {
                            TextField("Kategorie Name", text: $newCategory)
                                .textFieldStyle(RoundedBorderTextFieldStyle())
                                .foregroundColor(MoNaldoColors.adaptiveText)
                                .background(
                                    RoundedRectangle(cornerRadius: 10)
                                        .fill(Color.white)
                                )
                            
                            Button(action: {
                                if !newCategory.isEmpty {
                                    categories.append(newCategory)
                                    newCategory = ""
                                    showingAddCategorySheet = false
                                }
                            }) {
                                Text("Kategorie hinzufügen")
                            }
                        }
                        .navigationTitle("Neue Kategorie")
                        .navigationBarItems(
                            trailing: Button("Abbrechen") {
                                newCategory = ""
                                showingAddCategorySheet = false
                            }
                        )
                    }
                }
                .sheet(isPresented: $showingIncomeCategorySheet) {
                    NavigationView {
                        List {
                            ForEach(incomeCategories, id: \.self) { category in
                                Button(action: {
                                    addIncome(category: category)
                                    showingIncomeCategorySheet = false
                                }) {
                                    Text(category)
                                        .foregroundColor(MoNaldoColors.adaptiveText)
                                }
                            }
                            .onDelete(perform: deleteIncomeCategory)
                            
                            Button(action: {
                                showingIncomeCategorySheet = false
                                showingAddIncomeCategorySheet = true
                            }) {
                                HStack {
                                    Image(systemName: "plus.circle.fill")
                                    Text("Neue Kategorie")
                                }
                                .foregroundColor(MoNaldoColors.adaptiveText)
                            }
                        }
                        .navigationTitle("Einnahme Kategorie")
                        .navigationBarItems(trailing: Button("Abbrechen") {
                            showingIncomeCategorySheet = false
                        })
                    }
                }
                .sheet(isPresented: $showingAddIncomeCategorySheet) {
                    NavigationView {
                        Form {
                            TextField("Kategorie Name", text: $newIncomeCategory)
                                .textFieldStyle(RoundedBorderTextFieldStyle())
                                .foregroundColor(MoNaldoColors.adaptiveText)
                                .background(
                                    RoundedRectangle(cornerRadius: 10)
                                        .fill(Color.white)
                                )
                            
                            Button(action: {
                                if !newIncomeCategory.isEmpty {
                                    incomeCategories.append(newIncomeCategory)
                                    newIncomeCategory = ""
                                    showingAddIncomeCategorySheet = false
                                }
                            }) {
                                Text("Kategorie hinzufügen")
                            }
                        }
                        .navigationTitle("Neue Einnahme-Kategorie")
                        .navigationBarItems(trailing: Button("Abbrechen") {
                            newIncomeCategory = ""
                            showingAddIncomeCategorySheet = false
                        })
                    }
                }
                .sheet(isPresented: $showingCurrencyPicker) {
                    NavigationView {
                        List(Currency.allCases, id: \.self) { currency in
                            Button(action: {
                                selectedCurrency = currency
                                UserDefaults.standard.set(currency.rawValue, forKey: currencyKey)
                                showingCurrencyPicker = false
                            }) {
                                HStack {
                                    Image(systemName: currency.icon)
                                        .foregroundColor(MoNaldoColors.adaptiveText)
                                        .font(.title2)
                                    
                                    VStack(alignment: .leading) {
                                        Text(currency.name)
                                            .foregroundColor(MoNaldoColors.adaptiveText)
                                        Text(currency.symbol)
                                            .foregroundColor(MoNaldoColors.adaptiveText)
                                            .font(.subheadline)
                                    }
                                    .padding(.leading, 8)
                                    
                                    Spacer()
                                    
                                    if currency == selectedCurrency {
                                        Image(systemName: "checkmark")
                                            .foregroundColor(MoNaldoColors.adaptiveText)
                                    }
                                }
                            }
                        }
                        .navigationTitle("Währung wählen")
                        .navigationBarItems(trailing: Button("Abbrechen") {
                            showingCurrencyPicker = false
                        })
                    }
                }
                .alert("Monatsgehalt ändern", isPresented: $showingSalaryAlert) {
                    Button("Abbrechen", role: .cancel) {
                        salary = currentMonthlySalary > 0 ? String(format: "%.2f", currentMonthlySalary) : ""
                    }
                    Button("Ändern") {
                        if let newSalary = Double(salary) {
                            if newSalary != currentMonthlySalary {
                                balance -= currentMonthlySalary
                                balance += newSalary
                                currentMonthlySalary = newSalary
                                UserDefaults.standard.set(newSalary, forKey: "monthlySalary")
                                saveData()
                            }
                        }
                    }
                } message: {
                    Text("Möchten Sie das Monatsgehalt wirklich auf \(salary) \(selectedCurrency.symbol) ändern?")
                }
                .alert("Achtung: Mindestbetrag Limit erreicht", isPresented: $showingMinimumBalanceAlert) {
                    Button("OK", role: .cancel) { }
                } message: {
                    Text("Der Kontostand nähert sich dem Mindestbetrag von \(minimumBalance, specifier: "%.2f") \(selectedCurrency.symbol)")
                }
                .onAppear {
                    loadData()
                }
            }
            .navigationViewStyle(DoubleColumnNavigationViewStyle()) // iPad specific
            .tabItem {
                Image(systemName: "house.fill")
                Text("Übersicht")
            }
            
            NavigationView {
                IncomeView(
                    incomeEntries: incomeEntries,
                    onDelete: deleteIncome,
                    currency: selectedCurrency
                )
            }
            .tabItem {
                Image(systemName: "arrow.down.circle.fill")
                Text("Einnahmen")
            }
            
            NavigationView {
                ExpensesView(
                    expenseEntries: expenseEntries,
                    onDelete: deleteExpense,
                    currency: selectedCurrency
                )
            }
            .tabItem {
                Image(systemName: "list.bullet")
                Text("Ausgaben")
            }
            
            NavigationView {
                CalendarView(
                    expenseEntries: expenseEntries,
                    incomeEntries: incomeEntries,
                    currency: selectedCurrency,
                    balance: balance
                )
            }
            .tabItem {
                Image(systemName: "calendar")
                Text("Kalender")
            }
        }
        .accentColor(MoNaldoColors.gold)
    }

    func addExpense(category: String) {
        if let value = Double(expense), !expenseName.isEmpty {
            let newExpense = ExpenseEntry(
                amount: value,
                name: expenseName,
                category: category,
                date: Date()
            )
            expenseEntries.append(newExpense)
            balance -= value
            saveData()
            checkMinimumBalance() // Check after expense
            expense = ""
            expenseName = ""
        }
    }

    func deleteExpense(_ expense: ExpenseEntry) {
        if let index = expenseEntries.firstIndex(where: { $0.id == expense.id }) {
            balance += expense.amount
            expenseEntries.remove(at: index)
            saveData()
        }
    }

    func deleteIncome(at index: Int) {
        let removedIncome = incomeEntries.remove(at: index)
        balance -= removedIncome.amount
        saveData()
        checkMinimumBalance() // Check after income deletion
    }

    func addIncome(category: String) {
        if let value = Double(income), !incomeName.isEmpty {
            let newIncome = IncomeEntry(
                amount: value,
                name: incomeName,
                category: category,
                date: Date()
            )
            incomeEntries.append(newIncome)
            balance += value
            saveData()
            income = ""
            incomeName = ""
        }
    }

    func deleteIncome(_ income: IncomeEntry) {
        if let index = incomeEntries.firstIndex(where: { $0.id == income.id }) {
            balance -= income.amount
            incomeEntries.remove(at: index)
            saveData()
            checkMinimumBalance() // Check after income deletion
        }
    }

    func saveData() {
        UserDefaults.standard.set(balance, forKey: balanceKey)
        UserDefaults.standard.set(categories, forKey: "savedCategories")
        UserDefaults.standard.set(incomeCategories, forKey: incomeCategoriesKey)
        
        if let encodedIncome = try? JSONEncoder().encode(incomeEntries) {
            UserDefaults.standard.set(encodedIncome, forKey: incomeEntriesKey)
        }
        if let encodedExpenses = try? JSONEncoder().encode(expenseEntries) {
            UserDefaults.standard.set(encodedExpenses, forKey: expenseEntriesKey)
        }
        UserDefaults.standard.set(minimumBalance, forKey: minimumBalanceKey)
        UserDefaults.standard.synchronize()
    }

    func loadData() {
        balance = UserDefaults.standard.double(forKey: balanceKey)
        currentMonthlySalary = UserDefaults.standard.double(forKey: "monthlySalary")
        salary = currentMonthlySalary > 0 ? String(format: "%.2f", currentMonthlySalary) : ""
        
        if let savedIncomeCategories = UserDefaults.standard.array(forKey: incomeCategoriesKey) as? [String] {
            incomeCategories = savedIncomeCategories
        }
        
        if let savedIncome = UserDefaults.standard.data(forKey: incomeEntriesKey) {
            if let decodedIncome = try? JSONDecoder().decode([IncomeEntry].self, from: savedIncome) {
                incomeEntries = decodedIncome
            }
        }
        
        if let savedExpenses = UserDefaults.standard.data(forKey: expenseEntriesKey) {
            if let decodedExpenses = try? JSONDecoder().decode([ExpenseEntry].self, from: savedExpenses) {
                expenseEntries = decodedExpenses
            }
        }
        
        if let savedCategories = UserDefaults.standard.array(forKey: "savedCategories") as? [String] {
            categories = savedCategories
        }
        
        if let savedCurrency = UserDefaults.standard.string(forKey: currencyKey),
           let currency = Currency(rawValue: savedCurrency) {
            selectedCurrency = currency
        }
        
        minimumBalance = UserDefaults.standard.double(forKey: minimumBalanceKey)
        checkMinimumBalance() // Check after loading data
    }

    private func formatDate(_ date: Date) -> String {
        let formatter = DateFormatter()
        formatter.dateStyle = .medium
        formatter.timeStyle = .short
        return formatter.string(from: date)
    }

    private func deleteCategory(at offsets: IndexSet) {
        categories.remove(atOffsets: offsets)
        saveData()
    }

    private func deleteIncomeCategory(at offsets: IndexSet) {
        incomeCategories.remove(atOffsets: offsets)
        saveData()
    }

    // Update checkMinimumBalance function
    private func checkMinimumBalance() {
        if minimumBalance > 0 && balance <= minimumBalance + 100 {
            showingMinimumBalanceAlert = true
        }
    }
}

struct SalaryInputView: View {
    @Binding var salary: String
    @Binding var currentMonthlySalary: Double
    @Binding var showingSalaryAlert: Bool
    @FocusState private var salaryFocused: Bool
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text("Monatsgehalt")
                .font(.subheadline)
                .foregroundColor(MoNaldoColors.adaptiveText)
            
            HStack {
                TextField("Monatsgehalt eingeben", text: $salary)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .keyboardType(.decimalPad)
                    .focused($salaryFocused)
                    .foregroundColor(MoNaldoColors.adaptiveText)
                    .background(
                        RoundedRectangle(cornerRadius: 10)
                            .fill(Color.white)
                    )
                    .onSubmit {
                        salaryFocused = false
                    }
                
                Button("Ändern") {
                    if let value = Double(salary) {
                        salaryFocused = false
                        showingSalaryAlert = true
                    }
                }
                .buttonStyle(CustomButtonStyle(backgroundColor: MoNaldoColors.darkBrown))
            }
        }
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 15)
                .fill(MoNaldoColors.adaptiveBackground)
                .overlay(
                    RoundedRectangle(cornerRadius: 15)
                        .stroke(MoNaldoColors.mint.opacity(0.2), lineWidth: 1)
                )
        )
        .padding(.horizontal)
    }
}

// Update CalendarView to use the actual balance
struct CalendarView: View {
    let expenseEntries: [ExpenseEntry]
    let incomeEntries: [IncomeEntry]
    let currency: Currency
    let balance: Double // Neue Property für den Kontostand
    @State private var selectedDate = Date()
    
    var body: some View {
        VStack {
            DatePicker(
                "Datum auswählen",
                selection: $selectedDate,
                displayedComponents: [.date]
            )
            .datePickerStyle(GraphicalDatePickerStyle())
            .environment(\.locale, Locale(identifier: "de_DE"))
            .padding()
            
            // Kontostand anzeigen
            HStack {
                Text("Verfügbares Geld")
                    .font(.headline)
                    .foregroundColor(MoNaldoColors.adaptiveText)
                
                Text(String(format: "%.2f %@", balance, currency.symbol))
                    .font(.title2.bold())
                    .foregroundColor(balance >= 0 ? .green : .red)
                    .shadow(color: (balance >= 0 ? Color.green : Color.red).opacity(0.3), radius: 2, x: 0, y: 0)
            }
            .padding()
            .background(
                RoundedRectangle(cornerRadius: 12)
                    .fill(MoNaldoColors.adaptiveSecondaryBackground)
                    .shadow(
                        color: Color.black.opacity(0.05),
                        radius: 10,
                        x: 0,
                        y: 5
                    )
            )
            .padding(.horizontal)
            .padding(.bottom)
            
            List {
                if expensesForSelectedDate().isEmpty && incomesForSelectedDate().isEmpty {
                    Text("Keine Transaktionen an diesem Tag")
                        .foregroundColor(MoNaldoColors.adaptiveText)
                } else {
                    // Einnahmen Section
                    if !incomesForSelectedDate().isEmpty {
                        Section(header: Text("Einnahmen")) {
                            ForEach(incomesForSelectedDate()) { income in
                                HStack {
                                    VStack(alignment: .leading) {
                                        Text(income.name)
                                            .font(.headline)
                                        Text(income.category)
                                            .font(.subheadline)
                                            .foregroundColor(MoNaldoColors.adaptiveSecondary)
                                    }
                                    
                                    Spacer()
                                    
                                    Text("+\(income.amount, specifier: "%.2f") \(currency.symbol)")
                                        .foregroundColor(.green)
                                }
                            }
                        }
                    }
                    
                    // Ausgaben Section
                    if !expensesForSelectedDate().isEmpty {
                        Section(header: Text("Ausgaben")) {
                            ForEach(expensesForSelectedDate()) { entry in
                                HStack {
                                    VStack(alignment: .leading) {
                                        Text(entry.name)
                                            .font(.headline)
                                        Text(entry.category)
                                            .font(.subheadline)
                                            .foregroundColor(MoNaldoColors.adaptiveSecondary)
                                    }
                                    
                                    Spacer()
                                    
                                    Text("-\(entry.amount, specifier: "%.2f") \(currency.symbol)")
                                        .foregroundColor(.red)
                                }
                            }
                        }
                    }
                }
            }
            .listStyle(InsetGroupedListStyle())
        }
        .navigationTitle("MoNaldo")
        .navigationBarTitleDisplayMode(.inline)
        .toolbarColorScheme(.dark, for: .navigationBar)
        .toolbarBackground(MoNaldoColors.darkBrown, for: .navigationBar)
        .toolbarBackground(.visible, for: .navigationBar)
    }
    
    private func expensesForSelectedDate() -> [ExpenseEntry] {
        let calendar = Calendar(identifier: .gregorian)
        return expenseEntries.filter { entry in
            calendar.isDate(entry.date, inSameDayAs: selectedDate)
        }
    }
    
    private func incomesForSelectedDate() -> [IncomeEntry] {
        let calendar = Calendar(identifier: .gregorian)
        return incomeEntries.filter { income in
            calendar.isDate(income.date, inSameDayAs: selectedDate)
        }
    }
    
    // Neue Funktion zur Berechnung des Kontostands für das ausgewählte Datum
    private func balanceForSelectedDate() -> Double {
        let calendar = Calendar(identifier: .gregorian)
        
        // Filtere alle Transaktionen bis zum ausgewählten Datum
        let expensesUntilDate = expenseEntries.filter { entry in
            calendar.compare(entry.date, to: selectedDate, toGranularity: .day) != .orderedDescending
        }
        
        let incomesUntilDate = incomeEntries.filter { entry in
            calendar.compare(entry.date, to: selectedDate, toGranularity: .day) != .orderedDescending
        }
        
        // Berechne den Kontostand
        let totalExpenses = expensesUntilDate.reduce(0) { $0 + $1.amount }
        let totalIncomes = incomesUntilDate.reduce(0) { $0 + $1.amount }
        
        return totalIncomes - totalExpenses
    }
}

// Add new MinimumBalanceInputView
struct MinimumBalanceInputView: View {
    @Binding var minimumBalance: Double
    let currency: Currency
    @State private var inputText: String = ""
    @FocusState private var isFocused: Bool
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text("Mindestbetrag")
                .font(.subheadline)
                .foregroundColor(MoNaldoColors.adaptiveText)
            
            HStack {
                TextField("Mindestbetrag eingeben", text: $inputText)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .keyboardType(.decimalPad)
                    .focused($isFocused)
                    .foregroundColor(MoNaldoColors.adaptiveText)
                    .onChange(of: inputText) { newValue in
                        if let value = Double(newValue) {
                            minimumBalance = value
                        }
                    }
                
                Text(currency.symbol)
                    .foregroundColor(MoNaldoColors.adaptiveText)
            }
        }
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 15)
                .fill(MoNaldoColors.adaptiveBackground)
                .overlay(
                    RoundedRectangle(cornerRadius: 15)
                        .stroke(MoNaldoColors.mint.opacity(0.2), lineWidth: 1)
                )
        )
        .padding(.horizontal)
        .onAppear {
            inputText = minimumBalance > 0 ? String(format: "%.2f", minimumBalance) : ""
        }
    }
}

#Preview {
    ContentView()
        .preferredColorScheme(.light) // Light Mode Preview
}

#Preview("Dark Mode") {
    ContentView()
        .preferredColorScheme(.dark)
}



