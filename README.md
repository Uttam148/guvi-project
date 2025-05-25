# guvi-project
student database

🔧 Step 1: Run This SQL First (in MySQL Workbench or CLI)

CREATE DATABASE studentdb;

USE studentdb;

CREATE TABLE students (
    id VARCHAR(10) PRIMARY KEY,
    name VARCHAR(50),
    age INT,
    course VARCHAR(50)
);

🧠 Step 2: Full Java Code (Single Block)

import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.sql.*;
import java.util.Vector;

// --- Model Class ---
class Student {
    private String id, name, course;
    private int age;

    public Student(String id, String name, int age, String course) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.course = course;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getCourse() { return course; }
}

// --- JDBC Utility ---
class DBUtil {
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/studentdb", "root", "your_password"
        );
    }
}

// --- DAO Class ---
class StudentDAO {
    public static boolean add(Student s) throws SQLException {
        String sql = "INSERT INTO students VALUES (?, ?, ?, ?)";
        try (Connection c = DBUtil.getConnection();
             PreparedStatement ps = c.prepareStatement(sql)) {
            ps.setString(1, s.getId());
            ps.setString(2, s.getName());
            ps.setInt(3, s.getAge());
            ps.setString(4, s.getCourse());
            return ps.executeUpdate() > 0;
        }
    }

    public static Student find(String id) throws SQLException {
        String sql = "SELECT * FROM students WHERE id = ?";
        try (Connection c = DBUtil.getConnection();
             PreparedStatement ps = c.prepareStatement(sql)) {
            ps.setString(1, id);
            ResultSet rs = ps.executeQuery();
            if (rs.next())
                return new Student(rs.getString(1), rs.getString(2), rs.getInt(3), rs.getString(4));
        }
        return null;
    }

    public static boolean update(Student s) throws SQLException {
        String sql = "UPDATE students SET name=?, age=?, course=? WHERE id=?";
        try (Connection c = DBUtil.getConnection();
             PreparedStatement ps = c.prepareStatement(sql)) {
            ps.setString(1, s.getName());
            ps.setInt(2, s.getAge());
            ps.setString(3, s.getCourse());
            ps.setString(4, s.getId());
            return ps.executeUpdate() > 0;
        }
    }

    public static boolean delete(String id) throws SQLException {
        String sql = "DELETE FROM students WHERE id=?";
        try (Connection c = DBUtil.getConnection();
             PreparedStatement ps = c.prepareStatement(sql)) {
            ps.setString(1, id);
            return ps.executeUpdate() > 0;
        }
    }

    public static Vector<Vector<Object>> getAllStudents() throws SQLException {
        Vector<Vector<Object>> data = new Vector<>();
        String sql = "SELECT * FROM students";
        try (Connection c = DBUtil.getConnection();
             Statement stmt = c.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            while (rs.next()) {
                Vector<Object> row = new Vector<>();
                row.add(rs.getString(1));
                row.add(rs.getString(2));
                row.add(rs.getInt(3));
                row.add(rs.getString(4));
                data.add(row);
            }
        }
        return data;
    }
}

// --- GUI Class ---
public class Main {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel("javax.swing.plaf.nimbus.NimbusLookAndFeel");
            } catch (Exception ignored) {}

            JFrame frame = new JFrame("Student Database");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(600, 500);
            frame.setResizable(false);

            JPanel formPanel = new JPanel(new GridBagLayout());
            formPanel.setBackground(Color.white);
            GridBagConstraints gbc = new GridBagConstraints();
            gbc.insets = new Insets(6, 6, 6, 6);
            gbc.fill = GridBagConstraints.HORIZONTAL;

            JTextField idField = new JTextField(15);
            JTextField nameField = new JTextField(15);
            JTextField ageField = new JTextField(15);
            JTextField courseField = new JTextField(15);

            int row = 0;
            gbc.gridx = 0; gbc.gridy = row; formPanel.add(new JLabel("ID:"), gbc);
            gbc.gridx = 1; formPanel.add(idField, gbc); row++;
            gbc.gridx = 0; gbc.gridy = row; formPanel.add(new JLabel("Name:"), gbc);
            gbc.gridx = 1; formPanel.add(nameField, gbc); row++;
            gbc.gridx = 0; gbc.gridy = row; formPanel.add(new JLabel("Age:"), gbc);
            gbc.gridx = 1; formPanel.add(ageField, gbc); row++;
            gbc.gridx = 0; gbc.gridy = row; formPanel.add(new JLabel("Course:"), gbc);
            gbc.gridx = 1; formPanel.add(courseField, gbc); row++;

            JPanel buttonPanel = new JPanel(new FlowLayout());
            JButton addBtn = new JButton("Add");
            JButton findBtn = new JButton("Search");
            JButton updateBtn = new JButton("Update");
            JButton deleteBtn = new JButton("Delete");
            JButton viewBtn = new JButton("View All");

            buttonPanel.add(addBtn);
            buttonPanel.add(findBtn);
            buttonPanel.add(updateBtn);
            buttonPanel.add(deleteBtn);
            buttonPanel.add(viewBtn);

            gbc.gridwidth = 2; gbc.gridx = 0; gbc.gridy = row;
            formPanel.add(buttonPanel, gbc);

            // Table
            String[] columnNames = { "ID", "Name", "Age", "Course" };
            JTable table = new JTable(new Vector<>(), new Vector<>(java.util.List.of(columnNames)));
            JScrollPane tableScroll = new JScrollPane(table);
            tableScroll.setPreferredSize(new Dimension(580, 200));

            // Container
            Container container = frame.getContentPane();
            container.setLayout(new BorderLayout());
            container.add(formPanel, BorderLayout.NORTH);
            container.add(tableScroll, BorderLayout.SOUTH);

            frame.setVisible(true);

            ActionListener validateAndExecute = e -> {
                String id = idField.getText().trim();
                String name = nameField.getText().trim();
                String ageStr = ageField.getText().trim();
                String course = courseField.getText().trim();

                if (id.isEmpty() || name.isEmpty() || ageStr.isEmpty() || course.isEmpty()) {
                    JOptionPane.showMessageDialog(frame, "All fields are required.");
                    return;
                }

                int age;
                try {
                    age = Integer.parseInt(ageStr);
                } catch (NumberFormatException ex) {
                    JOptionPane.showMessageDialog(frame, "Age must be a number.");
                    return;
                }

                Student s = new Student(id, name, age, course);
                try {
                    if (e.getSource() == addBtn) {
                        if (StudentDAO.add(s))
                            JOptionPane.showMessageDialog(frame, "Student added.");
                        else
                            JOptionPane.showMessageDialog(frame, "Add failed (duplicate ID?).");
                    } else if (e.getSource() == updateBtn) {
                        if (StudentDAO.update(s))
                            JOptionPane.showMessageDialog(frame, "Student updated.");
                        else
                            JOptionPane.showMessageDialog(frame, "Update failed.");
                    }
                } catch (Exception ex) {
                    JOptionPane.showMessageDialog(frame, "Error: " + ex.getMessage());
                }
            };

            addBtn.addActionListener(validateAndExecute);
            updateBtn.addActionListener(validateAndExecute);

            findBtn.addActionListener(e -> {
                try {
                    Student s = StudentDAO.find(idField.getText().trim());
                    if (s != null) {
                        nameField.setText(s.getName());
                        ageField.setText(String.valueOf(s.getAge()));
                        courseField.setText(s.getCourse());
                    } else {
                        JOptionPane.showMessageDialog(frame, "Student not found.");
                    }
                } catch (Exception ex) {
                    JOptionPane.showMessageDialog(frame, "Error: " + ex.getMessage());
                }
            });

            deleteBtn.addActionListener(e -> {
                try {
                    if (StudentDAO.delete(idField.getText().trim()))
                        JOptionPane.showMessageDialog(frame, "Student deleted.");
                    else
                        JOptionPane.showMessageDialog(frame, "Delete failed.");
                } catch (Exception ex) {
                    JOptionPane.showMessageDialog(frame, "Error: " + ex.getMessage());
                }
            });

            viewBtn.addActionListener(e -> {
                try {
                    Vector<Vector<Object>> data = StudentDAO.getAllStudents();
                    Vector<String> headers = new Vector<>(java.util.List.of(columnNames));
                    table.setModel(new javax.swing.table.DefaultTableModel(data, headers));
                } catch (Exception ex) {
                    JOptionPane.showMessageDialog(frame, "Error loading data: " + ex.getMessage());
                }
            });
        });
    }
}


