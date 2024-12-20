# ExpenseTracker
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;

public class ExpenseTracker extends JFrame implements ActionListener {
    private JTextField descriptionField, amountField;
    private JComboBox<String> categoryChoice;
    private JButton addButton, viewButton;
    private JLabel messageLabel, totalLabel;
    private String csvFilePath = "Expenses.csv";

    public ExpenseTracker() {
        setLayout(new FlowLayout());
        setTitle("Expense Tracker");
        setSize(600, 500);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        // Description
        add(new JLabel("Description:"));
        descriptionField = new JTextField(20);
        add(descriptionField);

        // Amount
        add(new JLabel("Amount:"));
        amountField = new JTextField(10);
        add(amountField);

        // Category
        add(new JLabel("Category:"));
        categoryChoice = new JComboBox<>(new String[]{"Food", "Transport", "Utilities", "Entertainment", "Other"});
        add(categoryChoice);

        // Add button
        addButton = new JButton("Add Expense");
        addButton.addActionListener(this);
        add(addButton);

        // View button
        viewButton = new JButton("View Expenses");
        viewButton.addActionListener(this);
        add(viewButton);

        // Message label
        messageLabel = new JLabel("");
        add(messageLabel);

        // Total label
        totalLabel = new JLabel("Total Expenses: $0.00");
        add(totalLabel);

        // Initialize CSV file if it doesn't exist
        initializeCSV();

        updateTotalExpenses(); // Update total expenses on startup

        setVisible(true);
    }

    private void initializeCSV() {
        File file = new File(csvFilePath);
        if (!file.exists()) {
            try (PrintWriter writer = new PrintWriter(new FileWriter(csvFilePath, true))) {
                writer.println("Date,Description,Category,Amount");
            } catch (IOException e) {
                e.printStackTrace();
                messageLabel.setText("Error initializing CSV file.");
            }
        }
    }
    @Override
    public void actionPerformed(ActionEvent e) {
        if (e.getSource() == addButton) {
            addExpense();
        } else if (e.getSource() == viewButton) {
            viewExpenses();
        }
    }
    private void addExpense() {
        String description = descriptionField.getText();
        String amount = amountField.getText();
        String category = (String) categoryChoice.getSelectedItem();
        String date = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
        if (description.isEmpty() || amount.isEmpty()) {
            messageLabel.setText("Please fill all fields.");
            return;
        }
        try {
            double amountValue = Double.parseDouble(amount);
            if (amountValue < 0) {
                messageLabel.setText("Amount cannot be negative.");
                return;
            }
            saveExpenseToCSV(date, description, category, amountValue);
            messageLabel.setText("Expense added successfully!");
            descriptionField.setText("");
            amountField.setText("");
            updateTotalExpenses(); // Update total expenses after adding
        } catch (NumberFormatException ex) {
            messageLabel.setText("Invalid amount. Please enter a valid number.");
        } catch (IOException ex) {
            messageLabel.setText("Error saving to CSV.");
            ex.printStackTrace();
        }
    }
    private void saveExpenseToCSV(String date, String description, String category, double amount) throws IOException {
        try (PrintWriter writer = new PrintWriter(new FileWriter(csvFilePath, true))) {
            writer.printf("%s,%s,%s,%.2f%n", date, description, category, amount);
        }
    }
    private void viewExpenses() {
        ArrayList<String> expenses = readExpensesFromCSV();
        if (expenses.isEmpty()) {
            JOptionPane.showMessageDialog(this, "No expenses recorded.", "View Expenses", JOptionPane.INFORMATION_MESSAGE);
            return;
        }

        StringBuilder expenseList = new StringBuilder("Expenses:\n");
        for (String expense : expenses) {
            expenseList.append(expense).append("\n");
        }
        JOptionPane.showMessageDialog(this, expenseList.toString(), "View Expenses", JOptionPane.INFORMATION_MESSAGE);
    }
    private ArrayList<String> readExpensesFromCSV() {
        ArrayList<String> expenses = new ArrayList<>();
        try (BufferedReader reader = new BufferedReader(new FileReader(csvFilePath))) {
            String line;
            reader.readLine(); // Skip header
            while ((line = reader.readLine()) != null) {
                expenses.add(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
            messageLabel.setText("Error reading from CSV.");
        }
        return expenses;
    }
    private void updateTotalExpenses() {
        double total = 0.0;
        ArrayList<String> expenses = readExpensesFromCSV();
        for (String expense : expenses) {
            String[] parts = expense.split(",");
            if (parts.length == 4) {
                total += Double.parseDouble(parts[3]);
            }
        }
        totalLabel.setText(String.format("Total Expenses: $%.2f", total));
    }
    public static void main(String[] args) {
        new ExpenseTracker();
    }
}
