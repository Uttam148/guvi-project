# guvi-project
student database
🔧 Step 1: Run This SQL First (in MySQL Workbench or CLI)
sql
Copy
Edit
CREATE DATABASE studentdb;

USE studentdb;

CREATE TABLE students (
    id VARCHAR(10) PRIMARY KEY,
    name VARCHAR(50),
    age INT,
    course VARCHAR(50)
);
🧠 Step 2: Full Java Code (Single Block)
java
Copy
Edit
// Save as Main.java

import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.sql.*;

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
            frame.setSize(500, 400);
            frame.setResizable(false);

            JPanel panel = new JPanel(new GridBagLayout());
            panel.setBackground(Color.white);
            GridBagConstraints gbc = new GridBagConstraints();
            gbc.insets = new Insets(8, 8, 8, 8);
            gbc.fill = GridBagConstraints.HORIZONTAL;

            JTextField idField = new JTextField(15);
            JTextField nameField = new JTextField(15);
            JTextField ageField = new JTextField(15);
            JTextField courseField = new JTextField(15);

            idField.setToolTipText("Enter Student ID");
            nameField.setToolTipText("Enter Student Name");
            ageField.setToolTipText("Enter Student Age");
            courseField.setToolTipText("Enter Course");

            JButton addBtn = new JButton("Add");
            JButton findBtn = new JButton("Search");
            JButton updateBtn = new JButton("Update");
            JButton deleteBtn = new JButton("Delete");

            int row = 0;
            gbc.gridx = 0; gbc.gridy = row; panel.add(new JLabel("ID:"), gbc);
            gbc.gridx = 1; panel.add(idField, gbc);
            row++;
            gbc.gridx = 0; gbc.gridy = row; panel.add(new JLabel("Name:"), gbc);
            gbc.gridx = 1; panel.add(nameField, gbc);
            row++;
            gbc.gridx = 0; gbc.gridy = row; panel.add(new JLabel("Age:"), gbc);
            gbc.gridx = 1; panel.add(ageField, gbc);
            row++;
            gbc.gridx = 0; gbc.gridy = row; panel.add(new JLabel("Course:"), gbc);
            gbc.gridx = 1; panel.add(courseField, gbc);
            row++;
            gbc.gridx = 0; gbc.gridy = row; panel.add(addBtn, gbc);
            gbc.gridx = 1; panel.add(findBtn, gbc);
            row++;
            gbc.gridx = 0; gbc.gridy = row; panel.add(updateBtn, gbc);
            gbc.gridx = 1; panel.add(deleteBtn, gbc);

            frame.add(panel);
            frame.setVisible(true);

            addBtn.addActionListener(e -> {
                try {
                    Student s = new Student(idField.getText(), nameField.getText(),
                            Integer.parseInt(ageField.getText()), courseField.getText());
                    if (StudentDAO.add(s))
                        JOptionPane.showMessageDialog(frame, "Student added.");
                    else
                        JOptionPane.showMessageDialog(frame, "Failed to add.");
                } catch (Exception ex) {
                    JOptionPane.showMessageDialog(frame, "Error: " + ex.getMessage());
                }
            });

            findBtn.addActionListener(e -> {
                try {
                    Student s = StudentDAO.find(idField.getText());
                    if (s != null) {
                        nameField.setText(s.getName());
                        ageField.setText(String.valueOf(s.getAge()));
                        courseField.setText(s.getCourse());
                    } else {
                        JOptionPane.showMessageDialog(frame, "Not found.");
                    }
                } catch (Exception ex) {
                    JOptionPane.showMessageDialog(frame, "Error: " + ex.getMessage());
                }
            });

            updateBtn.addActionListener(e -> {
                try {
                    Student s = new Student(idField.getText(), nameField.getText(),
                            Integer.parseInt(ageField.getText()), courseField.getText());
                    if (StudentDAO.update(s))
                        JOptionPane.showMessageDialog(frame, "Updated.");
                    else
                        JOptionPane.showMessageDialog(frame, "Not found.");
                } catch (Exception ex) {
                    JOptionPane.showMessageDialog(frame, "Error: " + ex.getMessage());
                }
            });

            deleteBtn.addActionListener(e -> {
                try {
                    if (StudentDAO.delete(idField.getText()))
                        JOptionPane.showMessageDialog(frame, "Deleted.");
                    else
                        JOptionPane.showMessageDialog(frame, "Not found.");
                } catch (Exception ex) {
                    JOptionPane.showMessageDialog(frame, "Error: " + ex.getMessage());
                }
            });
        });
    }
} 
