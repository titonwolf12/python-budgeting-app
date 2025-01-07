# python-budgeting-app
# this is a budgeting app I created to sharpen my python skills and calculate monthly budget, expenses, leftover, and % of saving goal covered per month
# .........................................................................................................................
import sys
from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QHBoxLayout, QLabel, QLineEdit, QPushButton, QTableWidget, QTableWidgetItem, QComboBox, QProgressBar, QDateEdit
from PyQt5.QtCore import Qt, QDate

class BudgetingApp(QWidget):
    def __init__(self):
        super().__init__()

        self.setWindowTitle("Perfect Budgeting App")
        self.setGeometry(100, 100, 800, 600)

        self.incomes = []  # List to store income details
        self.expenses = []  # List to store expense details
        self.goal = 0  # Savings goal

        # Main layout
        self.layout = QVBoxLayout()

        # Header
        self.header_label = QLabel("<b>Personal Budgeting App</b>")
        self.header_label.setAlignment(Qt.AlignCenter)
        self.header_label.setStyleSheet("font-size: 24px; color: #007BFF; font-weight: bold;")
        self.layout.addWidget(self.header_label)

        # Income Section
        self.income_label = QLabel("<b>Income</b>")
        self.layout.addWidget(self.income_label)

        # Income Table
        self.income_table = QTableWidget()
        self.income_table.setColumnCount(3)  # Source, Amount, Cycle
        self.income_table.setHorizontalHeaderLabels(["Source", "Amount ($)", "Pay Cycle"])
        self.layout.addWidget(self.income_table)

        # Add Income Button
        self.add_income_button = QPushButton("Add Income")
        self.add_income_button.clicked.connect(self.add_income)
        self.layout.addWidget(self.add_income_button)

        # Expense Section
        self.expenses_label = QLabel("<b>Expenses</b>")
        self.layout.addWidget(self.expenses_label)

        # Expenses Table
        self.expenses_table = QTableWidget()
        self.expenses_table.setColumnCount(5)  # Category, Amount, Recurrence, Start Date, Recurrence Type
        self.expenses_table.setHorizontalHeaderLabels(["Category", "Amount ($)", "Recurring (Y/N)", "Start Date", "Recurrence Type"])
        self.layout.addWidget(self.expenses_table)

        # Add Expense Button
        self.add_expense_button = QPushButton("Add Expense")
        self.add_expense_button.clicked.connect(self.add_expense)
        self.layout.addWidget(self.add_expense_button)

        # Goal Section
        self.goal_label = QLabel("<b>Set Financial Goal</b>")
        self.layout.addWidget(self.goal_label)

        self.goal_input = QLineEdit()
        self.goal_input.setPlaceholderText("Enter your savings goal ($)")
        self.layout.addWidget(self.goal_input)

        self.set_goal_button = QPushButton("Set Goal")
        self.set_goal_button.clicked.connect(self.set_goal)
        self.layout.addWidget(self.set_goal_button)

        # Calculate Remaining Balance
        self.calculate_button = QPushButton("Calculate Remaining Balance")
        self.calculate_button.clicked.connect(self.calculate_balance)
        self.layout.addWidget(self.calculate_button)

        # Remaining Balance Display
        self.balance_label = QLabel("<b>Remaining Balance: $0.00</b>")
        self.layout.addWidget(self.balance_label)

        # Financial Progress Bar
        self.progress_bar = QProgressBar()
        self.progress_bar.setMaximum(100)
        self.progress_bar.setValue(0)
        self.layout.addWidget(self.progress_bar)

        # Set the layout for the window
        self.setLayout(self.layout)

    def add_income(self):
        row_position = self.income_table.rowCount()
        self.income_table.insertRow(row_position)

        source_input = QLineEdit()
        source_input.setPlaceholderText("e.g., Salary")
        self.income_table.setCellWidget(row_position, 0, source_input)

        amount_input = QLineEdit()
        amount_input.setPlaceholderText("e.g., 3000")
        self.income_table.setCellWidget(row_position, 1, amount_input)

        pay_cycle_input = QComboBox()
        pay_cycle_input.addItems(["Daily", "Weekly", "Bi-Weekly", "Monthly", "Yearly"])
        self.income_table.setCellWidget(row_position, 2, pay_cycle_input)

    def add_expense(self):
        row_position = self.expenses_table.rowCount()
        self.expenses_table.insertRow(row_position)

        category_input = QLineEdit()
        category_input.setPlaceholderText("e.g., Rent")
        self.expenses_table.setCellWidget(row_position, 0, category_input)

        amount_input = QLineEdit()
        amount_input.setPlaceholderText("e.g., 1000")
        self.expenses_table.setCellWidget(row_position, 1, amount_input)

        recurrence_input = QComboBox()
        recurrence_input.addItems(["Yes", "No"])
        self.expenses_table.setCellWidget(row_position, 2, recurrence_input)

        start_date_input = QDateEdit(QDate.currentDate())
        self.expenses_table.setCellWidget(row_position, 3, start_date_input)

        recurrence_type_input = QComboBox()
        recurrence_type_input.addItems(["Daily", "Weekly", "Monthly", "Yearly"])
        self.expenses_table.setCellWidget(row_position, 4, recurrence_type_input)

    def set_goal(self):
        goal = self.goal_input.text().strip()  # Clean the input to remove extra spaces
        if goal.isdigit() and int(goal) > 0:
            self.goal = int(goal)
            self.goal_input.clear()  # Clear the input field
            self.update_progress()
        else:
            self.balance_label.setText("Invalid goal amount. Please enter a valid number.")

    def calculate_balance(self):
        try:
            # Calculate total income (converted to monthly)
            total_income = 0.0
            for row in range(self.income_table.rowCount()):
                amount_input = self.income_table.cellWidget(row, 1)
                if amount_input:
                    amount_text = amount_input.text().strip()
                    if amount_text.replace('.', '', 1).isdigit():  # Validate if the amount is a valid number
                        pay_cycle_input = self.income_table.cellWidget(row, 2)
                        pay_cycle = pay_cycle_input.currentText()

                        # Adjust income based on pay cycle
                        if pay_cycle == "Weekly":
                            total_income += float(amount_text) * 4  # Monthly equivalent of weekly income
                        elif pay_cycle == "Bi-Weekly":
                            total_income += float(amount_text) * 2  # Monthly equivalent of bi-weekly income
                        elif pay_cycle == "Daily":
                            total_income += float(amount_text) * 30  # Monthly equivalent of daily income
                        elif pay_cycle == "Yearly":
                            total_income += float(amount_text) / 12  # Monthly equivalent of yearly income
                        else:
                            total_income += float(amount_text)  # Monthly income remains the same

            # Calculate total expenses (converted to monthly)
            total_expenses = 0.0
            for row in range(self.expenses_table.rowCount()):
                amount_input = self.expenses_table.cellWidget(row, 1)
                if amount_input:
                    amount_text = amount_input.text().strip()
                    if amount_text.replace('.', '', 1).isdigit():  # Validate if the amount is a valid number
                        # Get the recurrence type
                        recurrence_type_input = self.expenses_table.cellWidget(row, 4)
                        recurrence_type = recurrence_type_input.currentText()

                        # Adjust the expense amount based on the recurrence type
                        if recurrence_type == "Weekly":
                            total_expenses += float(amount_text) * 4  # Monthly equivalent of weekly expense
                        elif recurrence_type == "Bi-Weekly":
                            total_expenses += float(amount_text) * 2  # Monthly equivalent of bi-weekly expense
                        elif recurrence_type == "Daily":
                            total_expenses += float(amount_text) * 30  # Monthly equivalent of daily expense
                        elif recurrence_type == "Yearly":
                            total_expenses += float(amount_text) / 12  # Monthly equivalent of yearly expense
                        else:
                            total_expenses += float(amount_text)  # Monthly expense remains the same

            # Calculate remaining balance
            remaining_balance = total_income - total_expenses
            self.balance_label.setText(f"<b>Remaining Balance: ${remaining_balance:.2f}</b>")
            self.update_progress()

        except Exception as e:
            print(f"Error calculating balance: {e}")
            self.balance_label.setText("<b>Remaining Balance: $0.00</b>")

    def update_progress(self):
        if self.goal > 0:
            current_balance = sum([float(self.income_table.cellWidget(row, 1).text()) for row in range(self.income_table.rowCount()) if self.income_table.cellWidget(row, 1).text().strip().replace('.', '', 1).isdigit()])
            progress_percentage = (current_balance / self.goal) * 100
            self.progress_bar.setValue(min(int(progress_percentage), 100))
        else:
            self.progress_bar.setValue(0)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = BudgetingApp()
    window.show()
    sys.exit(app.exec_())
