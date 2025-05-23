# guvi-project
student database
/*
 * Project: File-Based Student Database
 * Description: Java project that performs CRUD operations on student data using file handling.
 * Components: Model, DAO, GUI
 */

// --- Student.java ---
package model;

public class Student {
    private String id;
    private String name;
    private int age;
    private String course;

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

    public void setName(String name) { this.name = name; }
    public void setAge(int age) { this.age = age; }
    public void setCourse(String course) { this.course = course; }

    @Override
    public String toString() {
        return id + "," + name + "," + age + "," + course;
    }

    public static Student fromString(String line) {
        String[] parts = line.split(",");
        return new Student(parts[0], parts[1], Integer.parseInt(parts[2]), parts[3]);
    }
}

// --- StudentFileManager.java ---
package dao;

import model.Student;
import java.io.*;
import java.util.*;

public class StudentFileManager {
    private static final String FILE_PATH = "students.txt";

    public static void addStudent(Student student) throws IOException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(FILE_PATH, true))) {
            writer.write(student.toString());
            writer.newLine();
        }
    }

    public static Student searchById(String id) throws IOException {
        try (BufferedReader reader = new BufferedReader(new FileReader(FILE_PATH))) {
            String line;
            while ((line = reader.readLine()) != null) {
                Student s = Student.fromString(line);
                if (s.getId().equals(id)) {
                    return s;
                }
            }
        }
        return null;
    }

    public static boolean updateStudent(Student updated) throws IOException {
        File inputFile = new File(FILE_PATH);
        File tempFile = new File("temp.txt");

        boolean updatedFlag = false;
        try (
            BufferedReader reader = new BufferedReader(new FileReader(inputFile));
            BufferedWriter writer = new BufferedWriter(new FileWriter(tempFile))
        ) {
            String line;
            while ((line = reader.readLine()) != null) {
                Student s = Student.fromString(line);
                if (s.getId().equals(updated.getId())) {
                    writer.write(updated.toString());
                    updatedFlag = true;
                } else {
                    writer.write(s.toString());
                }
                writer.newLine();
            }
        }
        inputFile.delete();
        tempFile.renameTo(inputFile);
        return updatedFlag;
    }

    public static boolean deleteStudent(String id) throws IOException {
        File inputFile = new File(FILE_PATH);
        File tempFile = new File("temp.txt");

        boolean deleted = false;
        try (
            BufferedReader reader = new BufferedReader(new FileReader(inputFile));
            BufferedWriter writer = new BufferedWriter(new FileWriter(tempFile))
        ) {
            String line;
            while ((line = reader.readLine()) != null) {
                Student s = Student.fromString(line);
                if (s.getId().equals(id)) {
                    deleted = true;
                    continue;
                }
                writer.write(s.toString());
                writer.newLine();
            }
        }
        inputFile.delete();
        tempFile.renameTo(inputFile);
        return deleted;
    }
}

// --- StudentUI.java ---
package ui;

import dao.StudentFileManager;
import model.Student;

import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.io.IOException;

public class StudentUI {
    public static void main(String[] args) {
        JFrame frame = new JFrame("Student Database");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 300);

        JPanel panel = new JPanel(new GridLayout(6, 2));
        JTextField idField = new JTextField();
        JTextField nameField = new JTextField();
        JTextField ageField = new JTextField();
        JTextField courseField = new JTextField();

        JButton addButton = new JButton("Add");
        JButton searchButton = new JButton("Search");
        JButton updateButton = new JButton("Update");
        JButton deleteButton = new JButton("Delete");

        panel.add(new JLabel("ID:")); panel.add(idField);
        panel.add(new JLabel("Name:")); panel.add(nameField);
        panel.add(new JLabel("Age:")); panel.add(ageField);
        panel.add(new JLabel("Course:")); panel.add(courseField);
        panel.add(addButton); panel.add(searchButton);
        panel.add(updateButton); panel.add(deleteButton);

        frame.add(panel);
        frame.setVisible(true);

        addButton.addActionListener(e -> {
            try {
                Student s = new Student(idField.getText(), nameField.getText(), Integer.parseInt(ageField.getText()), courseField.getText());
                StudentFileManager.addStudent(s);
                JOptionPane.showMessageDialog(frame, "Student added.");
            } catch (Exception ex) {
                JOptionPane.showMessageDialog(frame, "Error: " + ex.getMessage());
            }
        });

        searchButton.addActionListener(e -> {
            try {
                Student s = StudentFileManager.searchById(idField.getText());
                if (s != null) {
                    nameField.setText(s.getName());
                    ageField.setText(String.valueOf(s.getAge()));
                    courseField.setText(s.getCourse());
                } else {
                    JOptionPane.showMessageDialog(frame, "Student not found.");
                }
            } catch (IOException ex) {
                JOptionPane.showMessageDialog(frame, "Error: " + ex.getMessage());
            }
        });

        updateButton.addActionListener(e -> {
            try {
                Student s = new Student(idField.getText(), nameField.getText(), Integer.parseInt(ageField.getText()), courseField.getText());
                if (StudentFileManager.updateStudent(s)) {
                    JOptionPane.showMessageDialog(frame, "Student updated.");
                } else {
                    JOptionPane.showMessageDialog(frame, "Student not found.");
                }
            } catch (IOException ex) {
                JOptionPane.showMessageDialog(frame, "Error: " + ex.getMessage());
            }
        });

        deleteButton.addActionListener(e -> {
            try {
                if (StudentFileManager.deleteStudent(idField.getText())) {
                    JOptionPane.showMessageDialog(frame, "Student deleted.");
                } else {
                    JOptionPane.showMessageDialog(frame, "Student not found.");
                }
            } catch (IOException ex) {
                JOptionPane.showMessageDialog(frame, "Error: " + ex.getMessage());
            }
        });
    }
}
