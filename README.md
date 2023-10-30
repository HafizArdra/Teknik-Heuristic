# Teknik-Heuristic

# 1. Pelajari class EightPuzzleSearch, EightPuzzleSpace, dan Node.

# 2. Ubahlah initial dan goal state dari program di atas sehingga bentuk initial dan goal statenya Gambar 5.8. Kemudian tentukan langkah-langkah mana saja sehingga puzzlenya mencapai goal state. Analisa dan bedakan dengan solusi pada point 1.

Buat Kodenya seperti ini

    package JavaApplivation2;
    import java.util.Vector;

    class Node {
    int[] state = new int[9];
    int cost;
    Node parent = null;
    Vector<Node> successors = new Vector<Node>();

    Node(int s[], Node parent) {
        this.parent = parent;
        for (int i = 0; i < 9; i++)
            state[i] = s[i];
    }

    public Node() {
    }

    public String toString() {
        String s = "";
        for (int i = 0; i < 9; i++) {
            s = s + state[i] + " ";
        }
        return s;
    }

    public boolean equals(Object node) {
        Node n = (Node) node;
        boolean result = true;
        for (int i = 0; i < 9; i++) {
            if (n.state[i] != state[i])
                result = false;
        }
        return result;
    }

    Vector<Node> getPath(Vector<Node> v) {
        v.insertElementAt(this, 0);
        if (parent != null)
            v = parent.getPath(v);
        return v;
    }

    Vector<Node> getPath() {
        return getPath(new Vector<Node>());
    }

    public static void main(String args[]) {
        new Node().run();
    }

    private void run() {
    }
    }

    class EightPuzzleSpace {
    Node getRoot() {
        int initialState[] = {3, 1, 2, 4, 7, 5, 6, 8, 0};
        return new Node(initialState, null);
    }

    Node getGoal() {
        int goalState[] = {1, 2, 3, 4, 0, 8, 5, 6, 7};
        return new Node(goalState, null);
    }

    Vector<Node> getSuccessors(Node parent) {
        Vector<Node> successors = new Vector<Node>();
        for (int r = 0; r < 3; r++) {
            for (int c = 0; c < 3; c++) {
                if (parent.state[(r * 3) + c] == 0) {
                    if (r > 0) {
                        successors.add(transformState(r - 1, c, r, c, parent));
                    }
                    if (r < 2) {
                        successors.add(transformState(r + 1, c, r, c, parent));
                    }
                    if (c > 0) {
                        successors.add(transformState(r, c - 1, r, c, parent));
                    }
                    if (c < 2) {
                        successors.add(transformState(r, c + 1, r, c, parent));
                    }
                }
            }
        }
        parent.successors = successors;
        return successors;
    }

    Node transformState(int r0, int c0, int r1, int c1, Node parent) {
        int[] s = parent.state;
        int[] newState = { s[0], s[1], s[2], s[3], s[4], s[5], s[6], s[7], s[8] };
        newState[(r1 * 3) + c1] = s[(r0 * 3) + c0];
        newState[(r0 * 3) + c0] = 0;
        return new Node(newState, parent);
    }

    public static void main(String args[]) {
        new EightPuzzleSpace().run();
    }

    private void run() {
    }
    }

    class EightPuzzleSearch {
    EightPuzzleSpace space = new EightPuzzleSpace();
    Vector<Node> open = new Vector<Node>();
    Vector<Node> closed = new Vector<Node>();

    int h1Cost(Node node) {
        int cost = 0;
        for (int i = 0; i < node.state.length; i++) {
            if (node.state[i] != i) cost++;
        }
        return cost;
    }

    int h2Cost(Node node) {
        int cost = 0;
        int state[] = node.state;
        for (int i = 0; i < state.length; i++) {
            int v0 = i, v1 = state[i];
            if (v1 == 0)
                continue;
            int row0 = v0 / 3, col0 = v0 % 3, row1 = v1 / 3, col1 = v1 % 3;
            int c = (Math.abs(row0 - row1) + Math.abs(col0 - col1));
            cost += c;
        }
        return cost;
    }

    /* Boleh diubah dengan memakai heuristic h1 atau h2 */
    int hCost(Node node) {
        return h1Cost(node);
    }

    Node getBestNode(Vector nodes) {
        int index = 0, minCost = Integer.MAX_VALUE;
        for (int i = 0; i < nodes.size(); i++) {
            Node node = (Node) nodes.elementAt(i);
            if (node.cost < minCost) {
                minCost = node.cost;
                index = i;
            }
        }
        Node bestNode = (Node) nodes.remove(index);
        return bestNode;
    }

    int getPreviousCost(Node node) {
        int i = open.indexOf(node);
        int cost = Integer.MAX_VALUE;
        if (i != -1) {
            cost = open.get(i).cost;
        } else {
            i = closed.indexOf(node);
            if (i != -1)
                cost = closed.get(i).cost;
        }
        return cost;
    }

    void printPath(Vector path) {
        for (int i = 0; i < path.size(); i++) {
            System.out.print(" " + path.elementAt(i) + "\n");
        }
    }

    void run() {
        Node root = space.getRoot();
        Node goal = space.getGoal();
        Node solution = null;
        open.add(root);
        System.out.print("\nRoot: " + root + "\n\n");
        while (open.size() > 0) {
            Node node = getBestNode(open);
            int pathLength = node.getPath().size();
            closed.add(node);
            if (node.equals(goal)) {
                solution = node;
                break;
            }
            Vector<Node> successors = space.getSuccessors(node);
            for (int i = 0; i < successors.size(); i++) {
                Node successor = successors.get(i);
                int cost = hCost(successor) + pathLength + 1;
                int previousCost;
                previousCost = getPreviousCost(successor);
                boolean inClosed;
                inClosed = closed.contains(successor);
                boolean inOpen = open.contains(successor);
                if (!(inClosed || inOpen) || cost < previousCost) {
                    if (inClosed)
                        closed.remove(successor);
                    if (!inOpen)
                        open.add(successor);
                    successor.cost = cost;
                    successor.parent = node;
                }
            }
        }
        if (solution != null) {
            Vector path = solution.getPath();
            System.out.print("\nSolution found\n");
            printPath(path);
        }
    }

    public static void main(String args[]) {
        new EightPuzzleSearch().run();
    }
    }

Dalam kode Eight Puzzle yang disediakan, langkah-langkah untuk mencapai keadaan tujuan dari keadaan awal terstruktur dalam class EightPuzzleSearch. Class ini bertanggung jawab atas penemuan solusi menggunakan algoritma pencarian A* dan memberikan fleksibilitas kepada pengguna untuk memilih heuristic yang akan digunakan, yaitu h1 atau h2. Proses pencarian solusi ini melibatkan beberapa tahap utama:

- Tahap pertama adalah menginisialisasi keadaan awal dan tujuan permainan Eight Puzzle.
- Setelah itu, node awal (root) dibuat dengan menggunakan keadaan awal tersebut.
- Node awal dimasukkan ke dalam daftar open.
  
Kemudian, selama daftar open belum kosong, pencarian berlanjut dengan langkah-langkah berikut:

a. Node dengan biaya terendah, yang dihitung berdasarkan heuristic dan panjang path, dipilih.

b. Node terpilih dimasukkan ke dalam daftar closed untuk menghindari pengulangan.

c. Jika node terpilih sama dengan keadaan tujuan, ini menandakan bahwa solusi telah ditemukan, dan pencarian selesai.

d. Selanjutnya, dilakukan pencarian node-node penerus dari node terpilih.

e. Untuk setiap node penerus, biaya baru dihitung berdasarkan heuristic, panjang path, dan biaya sebelumnya.

f. Jika node penerus belum pernah ada di daftar open atau memiliki biaya lebih rendah, node penerus tersebut dimasukkan ke dalam daftar open dengan biaya yang telah diperbarui.

Proses ini akan terus diulang hingga solusi ditemukan atau semua kemungkinan solusi telah dieksplorasi. Hasil akhir dari pencarian akan menghasilkan langkah-langkah solusi yang akan dicetak ke dalam output. Penting untuk diperhatikan bahwa dalam kerja sama antara class EightPuzzleSearch, EightPuzzleSpace, dan Node, EightPuzzleSearch bertanggung jawab atas pengaturan dan logika pencarian secara keseluruhan, EightPuzzleSpace memberikan informasi tentang keadaan awal dan tujuan Eight Puzzle serta menghasilkan node-node awal, sementara Node digunakan untuk merepresentasikan setiap keadaan dalam permainan Eight Puzzle dan menyimpan informasi penting mengenai keadaan tersebut, biaya yang terkait, serta node induknya. Semua komponen ini bekerja bersama-sama untuk mencapai solusi Eight Puzzle.

# 3. Ubahlah initial dan goal state dari program di atas sehingga bentuk initial dan goal statenya Gambar 5.9. Kemudian tentukan langkah-langkah mana saja sehingga puzzlenya mencapai goal state. Analisa dan bedakan dengan solusi pada point 1 dan 2.

Buat Kodenya seperti ini

    package JavaApplivation2;
    import java.util.Vector;

    class Node {
    int[] state = new int[9];
    int cost;
    Node parent = null;
    Vector<Node> successors = new Vector<Node>();

    Node(int s[], Node parent) {
        this.parent = parent;
        for (int i = 0; i < 9; i++)
            state[i] = s[i];
    }

    public Node() {
    }

    public String toString() {
        String s = "";
        for (int i = 0; i < 9; i++) {
            s = s + state[i] + " ";
        }
        return s;
    }

    public boolean equals(Object node) {
        Node n = (Node) node;
        boolean result = true;
        for (int i = 0; i < 9; i++) {
            if (n.state[i] != state[i])
                result = false;
        }
        return result;
    }

    Vector<Node> getPath(Vector<Node> v) {
        v.insertElementAt(this, 0);
        if (parent != null)
            v = parent.getPath(v);
        return v;
    }

    Vector<Node> getPath() {
        return getPath(new Vector<Node>());
    }
    }

    class EightPuzzleSpace {
    Node getRoot() {
        int initialState[] ={1, 5, 3, 4, 6, 8, 2, 7, 0};
        return new Node(initialState, null);
    }

    Node getGoal() {
        int goalState[] = {7, 6, 5, 8, 0, 4, 1, 2, 3};
        return new Node(goalState, null);
    }

    Vector<Node> getSuccessors(Node parent) {
        Vector<Node> successors = new Vector<Node>();
        for (int r = 0; r < 3; r++) {
            for (int c = 0; c < 3; c++) {
                if (parent.state[(r * 3) + c] == 0) {
                    if (r > 0) {
                        successors.add(transformState(r - 1, c, r, c, parent));
                    }
                    if (r < 2) {
                        successors.add(transformState(r + 1, c, r, c, parent));
                    }
                    if (c > 0) {
                        successors.add(transformState(r, c - 1, r, c, parent));
                    }
                    if (c < 2) {
                        successors.add(transformState(r, c + 1, r, c, parent));
                    }
                }
            }
        }
        parent.successors = successors;
        return successors;
    }

    Node transformState(int r0, int c0, int r1, int c1, Node parent) {
        int[] s = parent.state;
        int[] newState = { s[0], s[1], s[2], s[3], s[4], s[5], s[6], s[7], s[8] };
        newState[(r1 * 3) + c1] = s[(r0 * 3) + c0];
        newState[(r0 * 3) + c0] = 0;
        return new Node(newState, parent);
    }
    
    public static void main(String args[]) {
        new EightPuzzleSpace().run();
    }

    private void run() {
    }
    }

    class EightPuzzleSearch {
    EightPuzzleSpace space = new EightPuzzleSpace();
    Vector<Node> open = new Vector<Node>();
    Vector<Node> closed = new Vector<Node>();

    int h1Cost(Node node) {
        int cost = 0;
        for (int i = 0; i < node.state.length; i++) {
        if (node.state[i] != i) cost++; }
        return cost;
        }

    int h2Cost(Node node) {
        int cost = 0;
        int state[] = node.state;
        for (int i = 0; i < state.length; i++) {
            int v0 = i, v1 = state[i];
            if (v1 == 0)
                continue;
            int row0 = v0 / 3, col0 = v0 % 3, row1 = v1 / 3, col1 = v1 % 3;
            int c = (Math.abs(row0 - row1) + Math.abs(col0 - col1));
            cost += c;
        }
        return cost;
    }
    /*boleh diubah dengan memakai heuristic h1 atau h2 */
    int hCost(Node node) {
    return h1Cost(node);
    }
    Node getBestNode(Vector nodes) {
        int index = 0, minCost = Integer.MAX_VALUE;
        for (int i = 0; i < nodes.size(); i++) {
            Node node = (Node)nodes.elementAt(i);
            if (node.cost < minCost) {
                minCost = node.cost;
                index = i;
            }
        }
        Node bestNode = (Node)nodes.remove(index);
        return bestNode;
    }

    int getPreviousCost(Node node) {
        int i = open.indexOf(node);
        int cost = Integer.MAX_VALUE;
        if (i != -1) {
            cost = open.get(i).cost;
        } else {
            i = closed.indexOf(node);
            if (i != -1)
                cost = closed.get(i).cost;
        }
        return cost;
    }

    void printPath(Vector path) {
        for (int i = 0; i < path.size(); i++) {
            System.out.print(" " + path.elementAt(i) + "\n");
        }
    }

    void run() {
        Node root = space.getRoot();
        Node goal = space.getGoal();
        Node solution = null;
        open.add(root);
        System.out.print("\nRoot: " + root + "\n\n");
        while (open.size() > 0) {
            Node node = getBestNode(open);
            int pathLength = node.getPath().size();
            closed.add(node);
            if (node.equals(goal)) {
                solution = node;
                break;
            }
            Vector<Node> successors = space.getSuccessors(node);
            for (int i = 0; i < successors.size(); i++) {
                Node successor = successors.get(i);
                int cost = hCost(successor) + pathLength + 1;
                int previousCost;
                previousCost = getPreviousCost(successor);
                boolean inClosed;
                inClosed = closed.contains(successor);
                boolean inOpen = open.contains(successor);
                if (!(inClosed || inOpen) || cost < previousCost) {
                    if (inClosed)
                        closed.remove(successor);
                    if (!inOpen)
                        open.add(successor);
                    successor.cost = cost;
                    successor.parent = node;
                }
            }
        }
        if (solution != null) {
            Vector path = solution.getPath();
            System.out.print("\nSolution found\n");
            printPath(path);
        }
    }

    public static void main(String args[]) {
        new EightPuzzleSearch().run();
    }
    }

Implementasi algoritma A* dalam menyelesaikan masalah 8-puzzle didiskusikan dalam kode di atas. Algoritma ini memungkinkan pengguna untuk memilih dua heuristik yang berbeda, yaitu h1Cost dan h2Cost, yang secara signifikan memengaruhi jalannya algoritma. Proses pencarian menuju keadaan tujuan dapat diuraikan sebagai berikut:

Pertama, kita memiliki kondisi awal permainan 8-puzzle, yang ditentukan sebagai berikut: {1, 5, 3, 4, 6, 8, 2, 7, 0}. Selain itu, kita memiliki kondisi tujuan yang harus dicapai: {7, 6, 5, 8, 0, 4, 1, 2, 3}.

Algoritma A* dimulai dengan menggunakan kondisi awal sebagai root node. Selanjutnya, algoritma mengevaluasi semua kemungkinan langkah yang dapat diambil dari kondisi awal, dengan memperhatikan posisi kotak kosong, dan menghitung biaya menggunakan salah satu heuristik, yaitu h1Cost atau h2Cost. Algoritma bertujuan untuk menemukan jalur paling efisien menuju keadaan tujuan dengan mempertimbangkan biaya total, yang mencakup biaya heuristik, jarak yang telah ditempuh, dan biaya sebelumnya.

Setiap langkah yang diambil dalam perjalanan menuju keadaan tujuan dipilih berdasarkan prioritas biaya terendah, yang sangat dipengaruhi oleh heuristik yang dipilih. Algoritma A* terus berjalan hingga mencapai keadaan tujuan yang telah ditentukan sebelumnya.

Penting untuk dicatat bahwa penggunaan heuristik h1Cost dan h2Cost akan memiliki dampak besar pada perhitungan biaya dan jalannya algoritma. Kedua heuristik ini dapat menghasilkan solusi yang berbeda dalam beberapa situasi. Heuristik h1Cost mengukur biaya dengan menghitung jumlah angka yang tidak berada pada posisi yang benar, sementara h2Cost menghitung biaya berdasarkan jarak Manhattan antara angka yang salah dengan posisi yang seharusnya. Sehingga, pemilihan heuristik akan memengaruhi bagaimana algoritma A* menjelajahi ruang pencarian untuk menemukan solusi

# 4.  Ubahlah initial dan goal state dari program di atas sehingga bentuk initial dan goal statenya Gambar 5.10. Kemudian tentukan langkah-langkah mana saja sehingga puzzlenya mencapai goal state. Analisa dan bedakan dengan solusi pada point 1, 2, dan 3.

Buat kodenya seperti ini

    package JavaApplivation2;
    import java.util.Vector;
  
    class Node {
      int[] state = new int[9];
      int cost;
      Node parent = null;
      Vector<Node> successors = new Vector<Node>();
  
      Node(int s[], Node parent) {
          this.parent = parent;
          for (int i = 0; i < 9; i++)
              state[i] = s[i];
      }
  
      public Node() {
      }
  
      public String toString() {
          String s = "";
          for (int i = 0; i < 9; i++) {
              s = s + state[i] + " ";
          }
          return s;
      }
  
      public boolean equals(Object node) {
          Node n = (Node) node;
          boolean result = true;
          for (int i = 0; i < 9; i++) {
              if (n.state[i] != state[i])
                  result = false;
          }
          return result;
      }
  
      Vector<Node> getPath(Vector<Node> v) {
          v.insertElementAt(this, 0);
          if (parent != null)
              v = parent.getPath(v);
          return v;
      }
  
      Vector<Node> getPath() {
          return getPath(new Vector<Node>());
      }
      public static void main(String args[]) {
          new Node().run();
      }
  
      private void run() {
      }
    }

    class EightPuzzleSpace {
      Node getRoot() {
          int initialState[] ={1, 2, 3, 4, 5, 6, 7, 8, 0};
          return new Node(initialState, null);
      }
  
      Node getGoal() {
          int goalState[] = {1, 2, 3, 4, 0, 5, 6, 7, 8};
          return new Node(goalState, null);
      }
  
      Vector<Node> getSuccessors(Node parent) {
          Vector<Node> successors = new Vector<Node>();
          for (int r = 0; r < 3; r++) {
              for (int c = 0; c < 3; c++) {
                  if (parent.state[(r * 3) + c] == 0) {
                      if (r > 0) {
                          successors.add(transformState(r - 1, c, r, c, parent));
                      }
                      if (r < 2) {
                          successors.add(transformState(r + 1, c, r, c, parent));
                      }
                      if (c > 0) {
                          successors.add(transformState(r, c - 1, r, c, parent));
                      }
                      if (c < 2) {
                          successors.add(transformState(r, c + 1, r, c, parent));
                      }
                  }
              }
          }
          parent.successors = successors;
          return successors;
      }
  
      Node transformState(int r0, int c0, int r1, int c1, Node parent) {
          int[] s = parent.state;
          int[] newState = { s[0], s[1], s[2], s[3], s[4], s[5], s[6], s[7], s[8] };
          newState[(r1 * 3) + c1] = s[(r0 * 3) + c0];
          newState[(r0 * 3) + c0] = 0;
          return new Node(newState, parent);
      }
      
      public static void main(String args[]) {
          new EightPuzzleSpace().run();
      }
  
      private void run() {
      }
      
    }
  
    class EightPuzzleSearch {
      EightPuzzleSpace space = new EightPuzzleSpace();
      Vector<Node> open = new Vector<Node>();
      Vector<Node> closed = new Vector<Node>();
  
      int h1Cost(Node node) {
          int cost = 0;
          for (int i = 0; i < node.state.length; i++) {
          if (node.state[i] != i) cost++; }
          return cost;
          }
  
      int h2Cost(Node node) {
          int cost = 0;
          int state[] = node.state;
          for (int i = 0; i < state.length; i++) {
              int v0 = i, v1 = state[i];
              if (v1 == 0)
                  continue;
              int row0 = v0 / 3, col0 = v0 % 3, row1 = v1 / 3, col1 = v1 % 3;
              int c = (Math.abs(row0 - row1) + Math.abs(col0 - col1));
              cost += c;
          }
          return cost;
      }
    /*boleh diubah dengan memakai heuristic h1 atau h2 */
      int hCost(Node node) {
      return h1Cost(node);
      }
      Node getBestNode(Vector nodes) {
          int index = 0, minCost = Integer.MAX_VALUE;
          for (int i = 0; i < nodes.size(); i++) {
              Node node = (Node)nodes.elementAt(i);
              if (node.cost < minCost) {
                  minCost = node.cost;
                  index = i;
              }
          }
          Node bestNode = (Node)nodes.remove(index);
          return bestNode;
      }
  
      int getPreviousCost(Node node) {
          int i = open.indexOf(node);
          int cost = Integer.MAX_VALUE;
          if (i != -1) {
              cost = open.get(i).cost;
          } else {
              i = closed.indexOf(node);
              if (i != -1)
                  cost = closed.get(i).cost;
          }
          return cost;
      }
  
      void printPath(Vector path) {
          for (int i = 0; i < path.size(); i++) {
              System.out.print(" " + path.elementAt(i) + "\n");
          }
      }
  
      void run() {
          Node root = space.getRoot();
          Node goal = space.getGoal();
          Node solution = null;
          open.add(root);
          System.out.print("\nRoot: " + root + "\n\n");
          while (open.size() > 0) {
              Node node = getBestNode(open);
              int pathLength = node.getPath().size();
              closed.add(node);
              if (node.equals(goal)) {
                  solution = node;
                  break;
              }
              Vector<Node> successors = space.getSuccessors(node);
              for (int i = 0; i < successors.size(); i++) {
                  Node successor = successors.get(i);
                  int cost = hCost(successor) + pathLength + 1;
                  int previousCost;
                  previousCost = getPreviousCost(successor);
                  boolean inClosed;
                  inClosed = closed.contains(successor);
                  boolean inOpen = open.contains(successor);
                  if (!(inClosed || inOpen) || cost < previousCost) {
                      if (inClosed)
                          closed.remove(successor);
                      if (!inOpen)
                          open.add(successor);
                      successor.cost = cost;
                      successor.parent = node;
                  }
              }
          }
          if (solution != null) {
              Vector path = solution.getPath();
              System.out.print("\nSolution found\n");
              printPath(path);
          }
      }
  
      public static void main(String args[]) {
          new EightPuzzleSearch().run();
      }
    }

Dalam "code 3," dijelaskan implementasi algoritma A* untuk menyelesaikan masalah 8-puzzle. Dalam konteks ini, pengguna memiliki dua pilihan heuristik, yaitu h1Cost dan h2Cost, yang akan memengaruhi jalannya algoritma. Heuristik digunakan untuk menghitung biaya yang terkait dengan setiap langkah dalam proses pencarian solusi.

Pada awalnya, terdapat kondisi awal permainan, yang disebut sebagai root, dengan urutan angka {1, 2, 3, 4, 5, 6, 7, 8, 0}. Tujuan yang harus dicapai adalah kondisi akhir, dengan urutan angka {1, 2, 3, 4, 0, 5, 6, 7, 8}. Algoritma A* dimulai dengan kondisi awal ini dan berupaya mencari langkah-langkah yang diperlukan untuk mencapai tujuan.

Selama proses pencarian, heuristik yang dipilih, baik h1Cost atau h2Cost, digunakan untuk menghitung biaya setiap langkah. Algoritma selalu memilih langkah yang memiliki biaya terendah, sesuai dengan heuristik yang sedang digunakan, untuk melanjutkan perjalanan menuju tujuan. Algoritma ini akan terus berjalan hingga mencapai kondisi tujuan yang telah ditentukan.

Perlu dicatat bahwa ketiga implementasi yang mungkin, termasuk "code 1" dan "code 2," menggunakan algoritma A* untuk menyelesaikan masalah 8-puzzle, tetapi penggunaan heuristik yang berbeda dapat menghasilkan solusi yang berbeda dalam hal urutan langkah dan waktu yang dibutuhkan. Hasil dari "code 3" akan memberikan langkah-langkah spesifik yang menuju ke kondisi akhir {1, 2, 3, 4, 0, 5, 6, 7, 8} berdasarkan pilihan heuristik (h1Cost atau h2Cost) yang digunakan. Langkah-langkah ini akan terdapat dalam output dari implementasi "code 3" setelah dieksekusi.

# 5. Ubahlah initial dan goal state dari program dan class-class di atas sehingga bentuk initial dan goal statenya Gambar 5.11. Kemudian tentukan langkah-langkah mana saja sehingga puzzlenya mencapai goal state.

Buat Kodenya seperti ini

    package JavaApplication2;
    import java.util.Vector;

    class Node {
    char[] state = new char[9];
    int cost;
    Node parent = null;
    Vector<Node> successors = new Vector<Node>();

    Node(char s[], Node parent) {
        this.parent = parent;
        for (int i = 0; i < 9; i++)
            state[i] = s[i];
    }

    public Node() {
    }

    public String toString() {
        String s = "";
        for (int i = 0; i < 9; i++) {
            s = s + state[i] + " ";
        }
        return s;
    }

    public boolean equals(Object node) {
        Node n = (Node) node;
        boolean result = true;
        for (int i = 0; i < 9; i++) {
            if (n.state[i] != state[i])
                result = false;
        }
        return result;
    }

    Vector<Node> getPath(Vector<Node> v) {
        v.insertElementAt(this, 0);
        if (parent != null)
            v = parent.getPath(v);
        return v;
    }

    Vector<Node> getPath() {
        return getPath(new Vector<Node>());
    }

    public static void main(String args[]) {
        new Node().run();
    }

    private void run() {
    }
    }

    class EightPuzzleSpace {
    Node getRoot() {
        char initialState[] = {'D', 'B', 'E', 'A', 'F', 'G', 'H', 'C', 'x'};
        return new Node(initialState, null);
    }

    Node getGoal() {
        char goalState[] = {'A', 'H', 'G', 'B', 'x', 'F', 'C', 'D', 'E'};
        return new Node(goalState, null);
    }

    Vector<Node> getSuccessors(Node parent) {
        Vector<Node> successors = new Vector<Node>();
        for (int r = 0; r < 3; r++) {
            for (int c = 0; c < 3; c++) {
                if (parent.state[(r * 3) + c] == 'x') {
                    if (r > 0) {
                        successors.add(transformState(r - 1, c, r, c, parent));
                    }
                    if (r < 2) {
                        successors.add(transformState(r + 1, c, r, c, parent));
                    }
                    if (c > 0) {
                        successors.add(transformState(r, c - 1, r, c, parent));
                    }
                    if (c < 2) {
                        successors.add(transformState(r, c + 1, r, c, parent));
                    }
                }
            }
        }
        parent.successors = successors;
        return successors;
    }

    Node transformState(int r0, int c0, int r1, int c1, Node parent) {
        char[] s = parent.state;
        char[] newState = {'x', 'x', 'x', 'x', 'x', 'x', 'x', 'x', 'x'};
        newState[(r1 * 3) + c1] = s[(r0 * 3) + c0];
        newState[(r0 * 3) + c0] = 'x';
        return new Node(newState, parent);
    }

    public static void main(String args[]) {
        new EightPuzzleSpace().run();
    }

    private void run() {
    }
    }

    class EightPuzzleSearch {
    EightPuzzleSpace space = new EightPuzzleSpace();
    Vector<Node> open = new Vector<Node>();
    Vector<Node> closed = new Vector<Node>();

    int h1Cost(Node node) {
        int cost = 0;
        for (int i = 0; i < node.state.length; i++) {
            if (node.state[i] != 'x')
                cost++;
        }
        return cost;
    }

    int h2Cost(Node node) {
        int cost = 0;
        char state[] = node.state;
        for (int i = 0; i < state.length; i++) {
            char v = state[i];
            if (v != 'x') {
                int row0 = i / 3, col0 = i % 3;
                int row1 = i / 3, col1 = i % 3;
                if (v != 'x') {
                    int c = (Math.abs(row0 - row1) + Math.abs(col0 - col1));
                    cost += c;
                }
            }
        }
        return cost;
    }

    int hCost(Node node) {
        return h1Cost(node);
    }

    Node getBestNode(Vector nodes) {
        int index = 0, minCost = Integer.MAX_VALUE;
        for (int i = 0; i < nodes.size(); i++) {
            Node node = (Node) nodes.elementAt(i);
            if (node.cost < minCost) {
                minCost = node.cost;
                index = i;
            }
        }
        Node bestNode = (Node) nodes.remove(index);
        return bestNode;
    }

    int getPreviousCost(Node node) {
        int i = open.indexOf(node);
        int cost = Integer.MAX_VALUE;
        if (i != -1) {
            cost = open.get(i).cost;
        } else {
            i = closed.indexOf(node);
            if (i != -1)
                cost = closed.get(i).cost;
        }
        return cost;
    }

    void printPath(Vector path) {
        for (int i = 0; i < path.size(); i++) {
            System.out.print(" " + path.elementAt(i) + "\n");
        }
    }

    void run() {
        Node root = space.getRoot();
        Node goal = space.getGoal();
        Node solution = null;
        open.add(root);
        System.out.print("\nRoot: " + root + "\n\n");
        while (open.size() > 0) {
            Node node = getBestNode(open);
            int pathLength = node.getPath().size();
            closed.add(node);
            if (node.equals(goal)) {
                solution = node;
                break;
            }
            Vector<Node> successors = space.getSuccessors(node);
            for (int i = 0; i < successors.size(); i++) {
                Node successor = successors.get(i);
                int cost = hCost(successor) + pathLength + 1;
                int previousCost;
                previousCost = getPreviousCost(successor);
                boolean inClosed;
                inClosed = closed.contains(successor);
                boolean inOpen = open.contains(successor);
                if (!(inClosed || inOpen) || cost < previousCost) {
                    if (inClosed)
                        closed.remove(successor);
                    if (!inOpen)
                        open.add(successor);
                    successor.cost = cost;
                    successor.parent = node;
                }
            }
        }
        if (solution != null) {
            Vector path = solution.getPath();
            System.out.print("\nSolution found\n");
            printPath(path);
        }
    }

    public static void main(String args[]) {
        new EightPuzzleSearch().run();
    }
    }

Dalam menentukan langkah-langkah untuk mencapai keadaan akhir dalam Code 4 yang digunakan untuk menyelesaikan puzzle, kita dapat mengimplementasikan algoritma pencarian seperti A* Search dan memanfaatkan dua heuristik, yaitu h1Cost dan h2Cost yang telah tersedia dalam kode tersebut. Langkah-langkah umum yang bisa diikuti adalah sebagai berikut:

Pertama, langkah awal adalah menginisialisasi node awal menggunakan keadaan awal dari puzzle. Node awal ini akan menjadi titik awal dalam proses pencarian solusi. Selanjutnya, kita perlu menyiapkan dua daftar, yaitu open list (daftar terbuka) dan closed list (daftar tertutup). Node awal akan ditempatkan di dalam open list untuk memulai proses pencarian.

Selama open list masih memiliki elemen di dalamnya, kita akan terus menjalankan langkah-langkah berikut:

Pertama, kita akan memilih node dengan biaya terendah dari open list. Biaya ini dihitung sebagai jumlah dari biaya heuristik dan panjang path (f = biaya heuristik + panjang path). Jika node yang dipilih adalah keadaan akhir (goal state), maka itu berarti puzzle telah selesai, dan proses pencarian dapat diakhiri.

Selanjutnya, kita akan menghasilkan semua node turunan dari node saat ini. Setiap node turunan akan dinilai berdasarkan biaya f yang dihitung menggunakan salah satu dari h1Cost atau h2Cost, serta panjang jalur dari node awal hingga node saat ini.

Jika node turunan sudah ada dalam open list dan biaya f yang baru lebih rendah daripada yang sebelumnya, maka kita akan memperbarui biaya dan node orang tua (parent) dari node tersebut. Hal yang sama berlaku jika node turunan sudah ada dalam closed list. Kita akan memperbarui biaya dan node orang tua dari node tersebut, dan selanjutnya memindahkan node dari closed list ke dalam open list jika biayanya lebih rendah.

Jika node turunan belum ada dalam kedua daftar (open list dan closed list), maka kita akan menambahkan node tersebut ke dalam open list untuk dieksplorasi lebih lanjut.

Terakhir, setelah semua langkah di atas dijalankan untuk node saat ini, kita akan menandai node tersebut sebagai telah dieksplorasi dengan memindahkannya dari open list ke dalam closed list. Dengan demikian, node ini tidak akan dievaluasi lagi dalam proses pencarian selanjutnya. Dengan mengikuti langkah-langkah ini, kita dapat menggunakan algoritma A* Search dengan heuristik h1Cost atau h2Cost untuk mencapai keadaan akhir yang diinginkan dalam permainan puzzle.
