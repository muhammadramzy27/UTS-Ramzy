1\. Implementasi Kode (MiniCompiler) Berikut adalah bagian kode yang
telah diperbarui sesuai dengan instruksi tugas (TUGAS 1, 2, dan 3):

import re

class MiniCompiler:
    def __init__(self, source, env):
        # TUGAS 1: Perbarui regex agar mengenali simbol '^'
        # Menambahkan \^ ke dalam list karakter operator
        self._tokens = iter(re.findall(r'[a-zA-Z_]\w*|\d+(?:\.\d+)?|[+*/()\-\^]', source) + ['?'])
        self._current = None
        self._env = env 
        self._temp_count = 0
        self.advance()

    def advance(self):
        try:
            self._current = next(self._tokens)
        except StopIteration:
            self._current = None

    def expect(self, expected):
        if self._current != expected and not (expected == "ID" and self._current.isalnum()):
            raise ParserError(f"Expected {expected}, found {self._current}")
        token = self._current
        self.advance()
        return token

    def factor(self):
        token = self._current
        if token is not None and token.replace('.', '', 1).isdigit():
            self.advance()
            return Num(float(token) if '.' in token else int(token))
        elif token and token.isalpha():
            if token not in self._env:
                raise ParserError(f"Semantic Error: Undefined variable '{token}'")
            self.advance()
            return Var(token)
        elif token == '(':
            self.advance()
            node = self.expr()
            self.expect(')')
            return node
        raise ParserError(f"Unexpected token: {token}")

    # TUGAS 2: Implementasikan fungsi power()
    def power(self):
        node = self.factor()
        while self._current == '^':
            op = self._current
            self.advance()
            # Pemanggilan rekursif ke power() atau factor() tergantung arah asosiativitas.
            # Umumnya pangkat adalah kanan-ke-kiri, tapi untuk tugas ini kita gunakan factor.
            node = BinOp(left=node, op=op, right=self.factor())
        return node

    def term(self):
        # TUGAS 3: Hubungkan hierarki ke self.power()
        # Sebelum perkalian/pembagian, kita evaluasi pangkat terlebih dahulu
        node = self.power() 
        while self._current in ('*', '/'):
            op = self._current
            self.advance()
            node = BinOp(left=node, op=op, right=self.power())
        return node

    def expr(self):
        node = self.term()
        while self._current in ('+', '-'):
            op = self._current
            self.advance()
            node = BinOp(left=node, op=op, right=self.term())
        return node

    def generate_tac(self, node):
        if isinstance(node, Num): return str(node.value)
        if isinstance(node, Var): return node.name
        
        left_val = self.generate_tac(node.left)
        right_val = self.generate_tac(node.right)
        
        self._temp_count += 1
        temp_name = f"t{self._temp_count}"
        print(f"{temp_name} = {left_val} {node.op} {right_val}")
        return temp_name

===========================================================================================================================================================
        
Berikut adalah output dari kode tersebut jika dijalankan dengan contoh
input yang ada pada tugas (a \^ 2 + b \* c):
Input: a ^ 2 + b * c

--- Output Three Address Code (TAC) ---
t1 = a ^ 2
t2 = b * c
t3 = t1 + t2

Penjelasan Output: t1 = a \^ 2: Operasi pangkat dikerjakan pertama kali
karena memiliki prioritas tertinggi (hasil dari fungsi power()).

t2 = b \* c: Operasi perkalian dikerjakan berikutnya (hasil dari fungsi
term()).

t3 = t1 + t2: Terakhir, hasil dari pangkat (t1) dijumlahkan dengan hasil
perkalian (t2) (hasil dari fungsi expr()).


===================================================================================================================================================================

4. Jawaban Pertanyaan Refleksi Berikut adalah penjelasan untuk bagian
refleksi:

(1.) Mengapa fungsi power() harus dipanggil di dalam term(), bukan
sebaliknya?

Jawaban: Hal ini dikarenakan prinsip Operator Precedence (Prioritas
Operator) dalam parsing. Dalam struktur top-down parser, fungsi yang
dipanggil lebih dalam (lebih jauh dari expr) memiliki prioritas yang
lebih tinggi. Karena pangkat (\^) secara matematika lebih kuat daripada
perkalian (\*), maka term() harus memanggil power() agar operasi pangkat
dikelompokkan terlebih dahulu di dalam pohon sintaks (AST) sebelum
diproses oleh operasi perkalian.

(2.)Apa yang terjadi pada fase Analisis Semantik jika variabel z
digunakan tetapi tidak ada di symbol_table?

Jawaban: Program akan melempar error (ParserError) dengan pesan
\"Semantic Error: Undefined variable \'z\'\". Pada fase analisis
semantik, kompilator bertugas memeriksa apakah simbol yang digunakan
sudah dideklarasikan atau tersedia dalam lingkup (scope) tersebut. Jika
tidak ada, proses kompilasi akan berhenti karena kompilator tidak
mengetahui nilai atau tipe data dari variabel tersebut.

(3.)Jelaskan mengapa dalam TAC, instruksi untuk a \^ 2 harus muncul
sebelum instruksi untuk +.

Jawaban: Three Address Code (TAC) merepresentasikan urutan eksekusi yang
linier. Karena prioritas operator pangkat lebih tinggi daripada
penjumlahan, nilai dari a \^ 2 harus dihitung terlebih dahulu dan
disimpan dalam variabel sementara (misal: t1). Hasil dari variabel
sementara tersebut kemudian baru bisa digunakan sebagai input untuk
operasi penjumlahan. Tanpa menghitung pangkat terlebih dahulu, operasi
penjumlahan tidak akan memiliki nilai operan yang valid untuk diproses.
