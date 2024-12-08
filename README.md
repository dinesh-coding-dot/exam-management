import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.ArrayList;

public class ExamManagementSystemGUI {
    // Question Class
    static class Question {
        private String questionText;
        private String[] options;
        private int correctAnswer;

        public Question(String questionText, String[] options, int correctAnswer) {
            this.questionText = questionText;
            this.options = options;
            this.correctAnswer = correctAnswer;
        }

        public String getQuestionText() {
            return questionText;
        }

        public String[] getOptions() {
            return options;
        }

        public int getCorrectAnswer() {
            return correctAnswer;
        }

        public String getDetails() {
            StringBuilder details = new StringBuilder("<html>Question: " + questionText + "<br>Options:<br>");
            for (int i = 0; i < options.length; i++) {
                details.append((i + 1)).append(". ").append(options[i]).append("<br>");
            }
            details.append("</html>");
            return details.toString();
        }
    }

    // Exam Class
    static class Exam {
        private String subject;
        private ArrayList<Question> questions;

        public Exam(String subject) {
            this.subject = subject;
            this.questions = new ArrayList<>();
        }

        public String getSubject() {
            return subject;
        }

        public ArrayList<Question> getQuestions() {
            return questions;
        }

        public void addQuestion(Question question) {
            questions.add(question);
        }

        public String getDetails() {
            return "<html>Subject: " + subject + "<br>Number of Questions: " + questions.size() + "</html>";
        }
    }

    private ArrayList<Exam> exams = new ArrayList<>();
    private JFrame frame;

    public ExamManagementSystemGUI() {
        // Main frame setup
        frame = new JFrame("Exam Management System");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(600, 400);

        // Main menu panel
        JPanel menuPanel = new JPanel();
        menuPanel.setLayout(new GridLayout(4, 1));

        JButton createExamButton = new JButton("Create Exam");
        JButton addQuestionButton = new JButton("Add Question to Exam");
        JButton viewExamsButton = new JButton("View Exams");
        JButton takeExamButton = new JButton("Take Exam");

        menuPanel.add(createExamButton);
        menuPanel.add(addQuestionButton);
        menuPanel.add(viewExamsButton);
        menuPanel.add(takeExamButton);

        frame.add(menuPanel);

        // Action Listeners for buttons
        createExamButton.addActionListener(e -> openCreateExamDialog());
        addQuestionButton.addActionListener(e -> openAddQuestionDialog());
        viewExamsButton.addActionListener(e -> openViewExamsDialog());
        takeExamButton.addActionListener(e -> openTakeExamDialog());

        frame.setVisible(true);
    }

    private void openCreateExamDialog() {
        JDialog dialog = new JDialog(frame, "Create Exam", true);
        dialog.setSize(400, 200);
        dialog.setLayout(new GridLayout(2, 2));

        JTextField subjectField = new JTextField();
        dialog.add(new JLabel("Subject:"));
        dialog.add(subjectField);

        JButton createButton = new JButton("Create");
        JButton cancelButton = new JButton("Cancel");

        dialog.add(createButton);
        dialog.add(cancelButton);

        createButton.addActionListener(e -> {
            String subject = subjectField.getText();
            if (!subject.isEmpty()) {
                exams.add(new Exam(subject));
                JOptionPane.showMessageDialog(dialog, "Exam created successfully!");
                dialog.dispose();
            } else {
                JOptionPane.showMessageDialog(dialog, "Subject cannot be empty!");
            }
        });

        cancelButton.addActionListener(e -> dialog.dispose());
        dialog.setVisible(true);
    }

    private void openAddQuestionDialog() {
        if (exams.isEmpty()) {
            JOptionPane.showMessageDialog(frame, "No exams available. Create an exam first!");
            return;
        }

        JDialog dialog = new JDialog(frame, "Add Question", true);
        dialog.setSize(500, 300);
        dialog.setLayout(new GridLayout(5, 2));

        JComboBox<String> examDropdown = new JComboBox<>();
        for (Exam exam : exams) {
            examDropdown.addItem(exam.getSubject());
        }

        JTextField questionField = new JTextField();
        JTextField optionsField = new JTextField();
        JTextField correctOptionField = new JTextField();

        dialog.add(new JLabel("Select Exam:"));
        dialog.add(examDropdown);
        dialog.add(new JLabel("Question:"));
        dialog.add(questionField);
        dialog.add(new JLabel("Options (comma-separated):"));
        dialog.add(optionsField);
        dialog.add(new JLabel("Correct Option Number:"));
        dialog.add(correctOptionField);

        JButton addButton = new JButton("Add");
        JButton cancelButton = new JButton("Cancel");

        dialog.add(addButton);
        dialog.add(cancelButton);

        addButton.addActionListener(e -> {
            int selectedExamIndex = examDropdown.getSelectedIndex();
            String questionText = questionField.getText();
            String[] options = optionsField.getText().split(",");
            int correctOption;

            try {
                correctOption = Integer.parseInt(correctOptionField.getText());
                if (correctOption < 1 || correctOption > options.length) {
                    throw new NumberFormatException();
                }
                exams.get(selectedExamIndex).addQuestion(new Question(questionText, options, correctOption));
                JOptionPane.showMessageDialog(dialog, "Question added successfully!");
                dialog.dispose();
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(dialog, "Invalid correct option number!");
            }
        });

        cancelButton.addActionListener(e -> dialog.dispose());
        dialog.setVisible(true);
    }

    private void openViewExamsDialog() {
        JDialog dialog = new JDialog(frame, "View Exams", true);
        dialog.setSize(500, 400);

        if (exams.isEmpty()) {
            dialog.add(new JLabel("No exams available."));
        } else {
            JPanel examPanel = new JPanel();
            examPanel.setLayout(new GridLayout(exams.size(), 1));

            for (Exam exam : exams) {
                JLabel examLabel = new JLabel(exam.getDetails());
                examPanel.add(examLabel);
            }

            dialog.add(new JScrollPane(examPanel));
        }

        JButton closeButton = new JButton("Close");
        dialog.add(closeButton, BorderLayout.SOUTH);
        closeButton.addActionListener(e -> dialog.dispose());
        dialog.setVisible(true);
    }

    private void openTakeExamDialog() {
        if (exams.isEmpty()) {
            JOptionPane.showMessageDialog(frame, "No exams available to take.");
            return;
        }

        JDialog dialog = new JDialog(frame, "Take Exam", true);
        dialog.setSize(500, 400);

        JComboBox<String> examDropdown = new JComboBox<>();
        for (Exam exam : exams) {
            examDropdown.addItem(exam.getSubject());
        }

        dialog.add(new JLabel("Select Exam:"));
        dialog.add(examDropdown, BorderLayout.NORTH);

        JButton startButton = new JButton("Start Exam");
        dialog.add(startButton, BorderLayout.SOUTH);

        startButton.addActionListener(e -> {
            int selectedExamIndex = examDropdown.getSelectedIndex();
            Exam selectedExam = exams.get(selectedExamIndex);
            dialog.dispose();

            int score = 0;
            for (Question question : selectedExam.getQuestions()) {
                String userAnswer = JOptionPane.showInputDialog(null, question.getDetails(), "Exam Question", JOptionPane.QUESTION_MESSAGE);
                if (userAnswer != null && userAnswer.equals(String.valueOf(question.getCorrectAnswer()))) {
                    score++;
                }
            }

            JOptionPane.showMessageDialog(frame, "You scored " + score + " out of " + selectedExam.getQuestions().size());
        });

        dialog.setVisible(true);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(ExamManagementSystemGUI::new);
    }
}
