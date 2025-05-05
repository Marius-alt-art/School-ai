# School-ai
Eine App die Schülern spielerisch das lernen beibringt
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.*;

public class LernApp extends JFrame {

    private JComboBox<String> fachWahl;
    private JTextArea frageFeld;
    private JTextField eingabeFeld;
    private JLabel feedbackLabel, punktestandLabel, levelLabel;
    private JButton absendenButton;

    private Map<String, Map<String, String>> fragenPool;
    private String aktuelleFrage, aktuellesFach;
    private int punkte = 0, level = 1;
    private List<String> fehlerFragen = new ArrayList<>();

    public LernApp() {
        setTitle("SmartTutor – Lernspiel mit KI-Logik");
        setSize(600, 400);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLayout(new BorderLayout());

        // GUI-Elemente
        frageFeld = new JTextArea();
        frageFeld.setEditable(false);
        frageFeld.setFont(new Font("Arial", Font.PLAIN, 16));
        frageFeld.setLineWrap(true);
        frageFeld.setWrapStyleWord(true);

        eingabeFeld = new JTextField();
        feedbackLabel = new JLabel("Antwort eingeben und absenden.");
        punktestandLabel = new JLabel("Punkte: 0");
        levelLabel = new JLabel("Level: 1");
        absendenButton = new JButton("Absenden");

        // Fächer-Dropdown
        String[] faecher = {"Mathe", "Deutsch", "Allgemeinwissen"};
        fachWahl = new JComboBox<>(faecher);
        fachWahl.addActionListener(e -> neuesFach((String) fachWahl.getSelectedItem()));

        // Panels
        JPanel topPanel = new JPanel(new GridLayout(1, 3));
        topPanel.add(fachWahl);
        topPanel.add(punktestandLabel);
        topPanel.add(levelLabel);

        JPanel untenPanel = new JPanel(new BorderLayout());
        untenPanel.add(eingabeFeld, BorderLayout.CENTER);
        untenPanel.add(absendenButton, BorderLayout.EAST);
        untenPanel.add(feedbackLabel, BorderLayout.SOUTH);

        add(topPanel, BorderLayout.NORTH);
        add(new JScrollPane(frageFeld), BorderLayout.CENTER);
        add(untenPanel, BorderLayout.SOUTH);

        // Fragen laden
        ladeFragen();
        neuesFach((String) fachWahl.getSelectedItem());

        absendenButton.addActionListener(e -> verarbeiteAntwort());
    }

    private void ladeFragen() {
        fragenPool = new HashMap<>();

        Map<String, String> mathe = new HashMap<>();
        mathe.put("Was ist 7 * 6?", "42");
        mathe.put("Was ist die Wurzel aus 81?", "9");
        mathe.put("Was ist 12 + 8?", "20");

        Map<String, String> deutsch = new HashMap<>();
        deutsch.put("Wer schrieb 'Faust'?", "Goethe");
        deutsch.put("Was ist ein Subjekt im Satz?", "Wer oder was");
        deutsch.put("Wie schreibt man 'viel' oder 'fiel'?", "viel");

        Map<String, String> wissen = new HashMap<>();
        wissen.put("Was ist die Hauptstadt von Frankreich?", "Paris");
        wissen.put("Wie viele Kontinente gibt es?", "7");
        wissen.put("Wie heißt das größte Meerestier?", "Blauwal");

        fragenPool.put("Mathe", mathe);
        fragenPool.put("Deutsch", deutsch);
        fragenPool.put("Allgemeinwissen", wissen);
    }

    private void neuesFach(String fach) {
        aktuellesFach = fach;
        neueFrage();
    }

    private void neueFrage() {
        Map<String, String> fragen = fragenPool.get(aktuellesFach);

        // Frage aus Fehlern bevorzugen
        if (!fehlerFragen.isEmpty() && Math.random() < 0.5) {
            aktuelleFrage = fehlerFragen.get((int) (Math.random() * fehlerFragen.size()));
        } else {
            aktuelleFrage = fragen.keySet().toArray(new String[0])[(int) (Math.random() * fragen.size())];
        }

        frageFeld.setText("Fach: " + aktuellesFach + "\n\nFrage: " + aktuelleFrage);
        eingabeFeld.setText("");
        feedbackLabel.setText("");
    }

    private void verarbeiteAntwort() {
        String eingabe = eingabeFeld.getText().trim();
        String richtigeAntwort = fragenPool.get(aktuellesFach).get(aktuelleFrage);

        if (eingabe.equalsIgnoreCase(richtigeAntwort)) {
            feedbackLabel.setText("Richtig!");
            punkte += 10;
            if (punkte >= level * 30) {
                level++;
            }
            fehlerFragen.remove(aktuelleFrage);
        } else {
            feedbackLabel.setText("Falsch. Richtig wäre: " + richtigeAntwort);
            if (!fehlerFragen.contains(aktuelleFrage)) {
                fehlerFragen.add(aktuelleFrage);
            }
        }

        punktestandLabel.setText("Punkte: " + punkte);
        levelLabel.setText("Level: " + level);

        Timer timer = new Timer(2000, e -> neueFrage());
        timer.setRepeats(false);
        timer.start();
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new LernApp().setVisible(true));
    }
}
