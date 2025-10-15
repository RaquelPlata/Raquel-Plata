/* RaquelApp.java
 *
 * Programa Java (Swing) que muestra un fondo pastel con corazones,
 * un avatar/emoji moreno (pelo corto negro, ojos marrones),
 * el nombre "Raquel Plata Ramos" y un mini-juego: hacer clic en corazones que caen para sumar puntos.
 *
 * Compilar:
 *   javac RaquelApp.java
 * Ejecutar:
 *   java RaquelApp
 *
 * Requiere Java 8+ (probado con Java 11/17).
 */

import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.awt.geom.*;
import java.util.*;
import java.util.List;

public class RaquelApp extends JPanel implements ActionListener, MouseListener {
    private final int WIDTH = 900;
    private final int HEIGHT = 420; // ratio similar a banner de GitHub
    private Timer timer;
    private List<Heart> hearts = new ArrayList<>();
    private Random rnd = new Random();
    private int ticks = 0;
    private int score = 0;
    private boolean running = true;

    // For clickable button-like areas
    private Rectangle restartRect = new Rectangle(10, HEIGHT - 40, 120, 30);

    // Emoji state
    private boolean emojiSmile = true;

    public RaquelApp() {
        setPreferredSize(new Dimension(WIDTH, HEIGHT));
        setBackground(Color.WHITE);
        timer = new Timer(20, this); // 50 FPS-ish
        timer.start();
        addMouseListener(this);
    }

    // Main paint
    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g.create();
        // Suavizado
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);

        // 1) Fondo degradado pastel (rosa -> lila -> celeste)
        GradientPaint gp = new GradientPaint(0, 0, new Color(255, 236, 239), WIDTH / 2f, HEIGHT, new Color(230, 235, 255));
        g2.setPaint(gp);
        g2.fillRect(0, 0, WIDTH, HEIGHT);

        // ligera textura de nubes (sutil)
        drawPastelClouds(g2);

        // 2) Dibujar corazones animados (detrás del texto principal para que se vean integrados)
        for (Heart h : hearts) {
            h.draw(g2);
        }

        // 3) Nombre centrado grande
        drawCenteredName(g2);

        // 4) Emoji/avatar (arriba-derecha)
        drawEmoji(g2, WIDTH - 170, 30, 120);

        // 5) UI del juego: puntuación, instrucciones y botón reiniciar
        drawGameUI(g2);

        g2.dispose();
    }

    private void drawPastelClouds(Graphics2D g2) {
        // Dibuja formas suaves tipo acuarela / bokeh muy sutil
        Composite orig = g2.getComposite();
        g2.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, 0.08f));
        for (int i = 0; i < 8; i++) {
            int w = 200 + rnd.nextInt(300);
            int h = 60 + rnd.nextInt(150);
            int x = rnd.nextInt(WIDTH);
            int y = rnd.nextInt(HEIGHT);
            Color c;
            switch (i % 3) {
                case 0: c = new Color(255, 230, 235); break; // rosa muy pálido
                case 1: c = new Color(240, 235, 255); break; // lila
                default: c = new Color(230, 245, 255); break; // celeste
            }
            g2.setPaint(new RadialGradientPaint(new Point2D.Float(x + w/2f, y + h/2f),
                    Math.max(w,h)/2f,
                    new float[]{0f,1f},
                    new Color[]{c, new Color(c.getRed(), c.getGreen(), c.getBlue(), 0)}));
            g2.fillRoundRect(x - w/4, y - h/4, w, h, 90, 90);
        }
        g2.setComposite(orig);
    }

    private void drawCenteredName(Graphics2D g2) {
        String name = "Raquel Plata Ramos";
        Font font = new Font("SansSerif", Font.BOLD, 36);
        g2.setFont(font);
        FontMetrics fm = g2.getFontMetrics();
        int textWidth = fm.stringWidth(name);
        int x = (WIDTH - textWidth) / 2;
        int y = HEIGHT / 2;

        // Sombra suave
        g2.setColor(new Color(0,0,0,40));
        g2.drawString(name, x+3, y+3);

        // Texto principal blanco con contorno pastel suave
        g2.setColor(Color.WHITE);
        g2.drawString(name, x, y);

        // Contorno suave con trazo ligero para hacerlo legible sobre el fondo
        g2.setStroke(new BasicStroke(2f));
        g2.setColor(new Color(255, 200, 220, 120));
        TextLayout tl = new TextLayout(name, font, g2.getFontRenderContext());
        Shape textShape = tl.getOutline(AffineTransform.getTranslateInstance(x, y));
        g2.draw(textShape);
    }

    private void drawEmoji(Graphics2D g2, int x, int y, int size) {
        // Parameters
        int cx = x + size/2;
        int cy = y + size/2;
        int radius = size/2;

        // Face color (morena ligera)
        Color skin = new Color(160, 110, 70);
        // Hair color (negro)
        Color hair = new Color(25, 25, 25);
        // Eye color marrón
        Color eye = new Color(95, 60, 30);

        // Fondo del avatar (círculo con sombra)
        g2.setColor(new Color(255,255,255,80));
        g2.fillOval(x-6, y-6, size+12, size+12);
        g2.setColor(new Color(0,0,0,40));
        g2.drawOval(x-6, y-6, size+12, size+12);

        // Cara
        g2.setColor(skin);
        g2.fillOval(x, y, size, size);

        // Pelo corto (semi-círculo arriba con recorte)
        g2.setColor(hair);
        Arc2D hairArc = new Arc2D.Double(x-2, y-10, size+4, size, 0, 180, Arc2D.CHORD);
        g2.fill(hairArc);

        // Pequeñas mechitas laterales (dar sensación de pelo corto)
        g2.fillOval(x + size - 24, y + 18, 18, 10);
        g2.fillOval(x + 6, y + 20, 18, 10);

        // Ojos
        int eyeW = size/7;
        int eyeH = size/9;
        int eyeY = y + size/2 - 8;
        int leftEyeX = x + size/3 - eyeW/2;
        int rightEyeX = x + 2*size/3 - eyeW/2;
        // sclera (blanco) - but subtle
        g2.setColor(new Color(255,255,255,230));
        g2.fillOval(leftEyeX, eyeY, eyeW, eyeH);
        g2.fillOval(rightEyeX, eyeY, eyeW, eyeH);
        // iris
        g2.setColor(eye);
        g2.fillOval(leftEyeX + 2, eyeY + 2, eyeW - 4, eyeH - 4);
        g2.fillOval(rightEyeX + 2, eyeY + 2, eyeW - 4, eyeH - 4);
        // brillo
        g2.setColor(new Color(255,255,255,200));
        g2.fillOval(leftEyeX + 4, eyeY + 3, 4, 4);
        g2.fillOval(rightEyeX + 4, eyeY + 3, 4, 4);

        // Nariz (sutil)
        g2.setColor(new Color(0,0,0,30));
        g2.fillOval(cx - 4, y + size/2 + 2, 8, 4);

        // Boca (sonrisa o neutra según emojiSmile)
        g2.setStroke(new BasicStroke(3f, BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND));
        g2.setColor(new Color(120, 40, 60));
        if (emojiSmile) {
            QuadCurve2D smile = new QuadCurve2D.Double(x + size*0.30, y + size*0.70,
                    cx, y + size*0.82,
                    x + size*0.70, y + size*0.70);
            g2.draw(smile);
        } else {
            g2.drawLine(x + size/3, y + size*3/4, x + size*2/3, y + size*3/4);
        }
    }

    private void drawGameUI(Graphics2D g2) {
        // Score
        g2.setFont(new Font("SansSerif", Font.BOLD, 16));
        g2.setColor(new Color(30, 30, 30));
        String scoreText = "Puntos: " + score;
        g2.drawString(scoreText, 10, 22);

        // Instrucciones
        g2.setFont(new Font("SansSerif", Font.PLAIN, 12));
        g2.setColor(new Color(30, 30, 30, 200));
        g2.drawString("Haz clic en los corazones que caen ♥ para sumar puntos. Haz clic en el avatar para cambiar su gesto.", 150, 22);

        // Reiniciar botón (simple)
        g2.setColor(new Color(255,255,255,200));
        g2.fill(restartRect);
        g2.setColor(new Color(200,150,180));
        g2.draw(restartRect);
        g2.setColor(new Color(80, 20, 60));
        g2.drawString("Reiniciar", restartRect.x + 18, restartRect.y + 20);

        // Pausa/Estado
        g2.setFont(new Font("SansSerif", Font.PLAIN, 12));
        String state = running ? "En marcha" : "Pausado";
        g2.setColor(new Color(30,30,30,160));
        g2.drawString("Estado: " + state, WIDTH - 120, HEIGHT - 10);
    }

    // Timer tick
    @Override
    public void actionPerformed(ActionEvent e) {
        if (!running) {
            repaint();
            return;
        }
        ticks++;
        // cada cierto número de ticks añadir un corazón
        if (ticks % 18 == 0) {
            spawnHeart();
        }

        // Mover corazones
        Iterator<Heart> it = hearts.iterator();
        while (it.hasNext()) {
            Heart h = it.next();
            h.update();
            if (h.y > HEIGHT + 60) {
                it.remove(); // se pierde
            }
        }
        repaint();
    }

    private void spawnHeart() {
        int size = 22 + rnd.nextInt(24);
        int x = 20 + rnd.nextInt(WIDTH - 60);
        int y = -30 - rnd.nextInt(80);
        float speed = 1f + rnd.nextFloat() * 2.2f;
        Color[] pal = {
                new Color(255, 170, 180),
                new Color(255, 220, 240),
                new Color(255, 200, 230),
                new Color(240, 200, 255),
                new Color(200, 230, 255)
        };
        Color c = pal[rnd.nextInt(pal.length)];
        hearts.add(new Heart(x, y, size, speed, c));
    }

    // Mouse interactions: clicks to collect hearts, toggle emoji, restart, pause
    @Override
    public void mouseClicked(MouseEvent e) {
        Point p = e.getPoint();
        // Check restart button
        if (restartRect.contains(p)) {
            score = 0;
            hearts.clear();
            running = true;
            return;
        }

        // Click on avatar (approx area)
        Rectangle avatarArea = new Rectangle(WIDTH - 170, 30, 120, 120);
        if (avatarArea.contains(p)) {
            emojiSmile = !emojiSmile;
            return;
        }

        // Click on hearts: find if clicked on top-most heart (reverse order)
        for (int i = hearts.size() - 1; i >= 0; i--) {
            Heart h = hearts.get(i);
            if (h.contains(p)) {
                hearts.remove(i);
                score += Math.max(1, 10 - (h.size / 6)); // puntuación según tamaño
                // pequeña "explosión" de partículas: spawn tiny hearts
                for (int k = 0; k < 6; k++) {
                    hearts.add(new Heart(h.x + rnd.nextInt(8) - 4, h.y, 6 + rnd.nextInt(6), 1.0f + rnd.nextFloat()*1.2f,
                            new Color(255, 120 + rnd.nextInt(120), 160)));
                }
                break;
            }
        }
    }

    // Otros métodos vacíos del listener
    @Override public void mousePressed(MouseEvent e) { }
    @Override public void mouseReleased(MouseEvent e) { }
    @Override public void mouseEntered(MouseEvent e) { }
    @Override public void mouseExited(MouseEvent e) { }

    // Heart inner class
    private class Heart {
        float x, y;
        int size;
        float speed;
        Color color;
        private Shape shapeCache = null;

        Heart(float x, float y, int size, float speed, Color color) {
            this.x = x; this.y = y; this.size = size; this.speed = speed; this.color = color;
        }

        void update() {
            // Movimiento: cae y tiene un ligero vaivén horizontal
            y += speed;
            x += Math.sin((y + x) / 25.0) * 0.8;
            // update shape cache
            shapeCache = null;
        }

        void draw(Graphics2D g2) {
            Shape heart = getShape();
            // sombra
            g2.setColor(new Color(0,0,0,30));
            AffineTransform at = AffineTransform.getTranslateInstance(2, 3);
            g2.fill(at.createTransformedShape(heart));
            // body
            g2.setColor(color);
            g2.fill(heart);
            g2.setColor(new Color(255,255,255,80));
            g2.draw(heart);
        }

        Shape getShape() {
            if (shapeCache != null) return shapeCache;
            // Construir un corazón centrado en (x,y) usando GeneralPath
            float w = size;
            float h = size;
            float cx = x;
            float cy = y;
            GeneralPath p = new GeneralPath();
            // Usamos dos círculos y un triángulo aproximado para la forma del corazón
            Ellipse2D left = new Ellipse2D.Float(cx - w * 0.4f, cy - h * 0.25f, w * 0.45f, h * 0.45f);
            Ellipse2D right = new Ellipse2D.Float(cx - w * 0.05f, cy - h * 0.25f, w * 0.45f, h * 0.45f);
            // Triángulo punto inferior
            Path2D triangle = new Path2D.Float();
            triangle.moveTo(cx - w * 0.45, cy);
            triangle.lineTo(cx + w * 0.5, cy);
            triangle.lineTo(cx, cy + h * 0.6);
            triangle.closePath();

            p.append(left, false);
            p.append(right, false);
            p.append(triangle, false);

            // Smooth by creating its outline (optional)
            shapeCache = p;
            return shapeCache;
        }

        boolean contains(Point pt) {
            Shape s = getShape();
            return s.contains(pt);
        }
    }

    // Main
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Portada de Raquel Plata Ramos - Pastel Hearts & Game");
            RaquelApp panel = new RaquelApp();

            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.getContentPane().add(panel);
            frame.pack();
            frame.setResizable(false);

            // Centrar ventana
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
