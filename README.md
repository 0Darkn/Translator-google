
Janela gráfica Qt

Botão Pasta de origem

Botão Pasta de destino

Seleção de língua origem e destino (todas as línguas)

Botão Traduzir

Botão Sair

Converte o texto para Markdown (.md)

Traduz todos os .txt da pasta


Usaremos a biblioteca PyQt5 para a interface e googletrans para traduzir usando o serviço do Google Translate.


---

1️⃣ Instalar as bibliotecas

No terminal:

pip install PyQt5 googletrans==4.0.0-rc1


---

2️⃣ Script completo — Tradutor Qt

# ============================================================
# Tradutor de ficheiros usando Google Translate
# Interface gráfica em Qt (PyQt5)
# Converte ficheiros .txt para Markdown (.md)
# ============================================================

import sys
import os

from PyQt5.QtWidgets import (
    QApplication, QWidget, QLabel, QPushButton,
    QFileDialog, QTextEdit, QComboBox, QVBoxLayout,
    QHBoxLayout
)

from googletrans import Translator, LANGUAGES


# ============================================================
# Classe principal da aplicação
# ============================================================

class TradutorApp(QWidget):

    def __init__(self):
        super().__init__()

        self.pasta_origem = ""
        self.pasta_destino = ""

        self.tradutor = Translator()

        self.init_ui()


# ============================================================
# Criar interface gráfica
# ============================================================

    def init_ui(self):

        self.setWindowTitle("Tradutor de Ficheiros para Markdown")

        layout = QVBoxLayout()

        # -------------------------
        # Seleção de línguas
        # -------------------------

        self.combo_origem = QComboBox()
        self.combo_destino = QComboBox()

        for codigo, nome in LANGUAGES.items():
            self.combo_origem.addItem(nome, codigo)
            self.combo_destino.addItem(nome, codigo)

        self.combo_origem.setCurrentText("english")
        self.combo_destino.setCurrentText("portuguese")

        layout.addWidget(QLabel("Língua origem"))
        layout.addWidget(self.combo_origem)

        layout.addWidget(QLabel("Língua destino"))
        layout.addWidget(self.combo_destino)

        # -------------------------
        # Botões
        # -------------------------

        botoes = QHBoxLayout()

        self.btn_origem = QPushButton("Pasta origem")
        self.btn_destino = QPushButton("Pasta destino")
        self.btn_traduzir = QPushButton("Traduzir")
        self.btn_sair = QPushButton("Sair")

        botoes.addWidget(self.btn_origem)
        botoes.addWidget(self.btn_destino)
        botoes.addWidget(self.btn_traduzir)
        botoes.addWidget(self.btn_sair)

        layout.addLayout(botoes)

        # -------------------------
        # Caixa de log
        # -------------------------

        self.log = QTextEdit()
        layout.addWidget(self.log)

        self.setLayout(layout)

        # -------------------------
        # Ligações
        # -------------------------

        self.btn_origem.clicked.connect(self.escolher_origem)
        self.btn_destino.clicked.connect(self.escolher_destino)
        self.btn_traduzir.clicked.connect(self.traduzir_pasta)
        self.btn_sair.clicked.connect(self.close)


# ============================================================
# Escolher pasta origem
# ============================================================

    def escolher_origem(self):

        pasta = QFileDialog.getExistingDirectory(self, "Selecionar pasta origem")

        if pasta:
            self.pasta_origem = pasta
            self.log.append(f"Pasta origem: {pasta}")


# ============================================================
# Escolher pasta destino
# ============================================================

    def escolher_destino(self):

        pasta = QFileDialog.getExistingDirectory(self, "Selecionar pasta destino")

        if pasta:
            self.pasta_destino = pasta
            self.log.append(f"Pasta destino: {pasta}")


# ============================================================
# Converter texto para Markdown
# ============================================================

    def texto_para_md(self, texto):

        linhas = texto.split("\n")

        md = []

        for linha in linhas:

            linha = linha.strip()

            if not linha:
                md.append("")
                continue

            # primeira linha vira título
            if len(md) == 0:
                md.append("# " + linha)

            else:
                md.append(linha)

        return "\n".join(md)


# ============================================================
# Traduzir todos os ficheiros
# ============================================================

    def traduzir_pasta(self):

        if not self.pasta_origem or not self.pasta_destino:
            self.log.append("Escolha as pastas primeiro.")
            return

        origem = self.combo_origem.currentData()
        destino = self.combo_destino.currentData()

        for ficheiro in os.listdir(self.pasta_origem):

            if not ficheiro.endswith(".txt"):
                continue

            caminho = os.path.join(self.pasta_origem, ficheiro)

            with open(caminho, "r", encoding="utf-8") as f:
                texto = f.read()

            self.log.append(f"Traduzindo: {ficheiro}")

            traduzido = self.tradutor.translate(
                texto,
                src=origem,
                dest=destino
            ).text

            md = self.texto_para_md(traduzido)

            nome_md = ficheiro.replace(".txt", ".md")

            destino_ficheiro = os.path.join(
                self.pasta_destino,
                nome_md
            )

            with open(destino_ficheiro, "w", encoding="utf-8") as f:
                f.write(md)

            self.log.append(f"Gravado: {nome_md}")

        self.log.append("Tradução terminada.")


# ============================================================
# Executar aplicação
# ============================================================

if __name__ == "__main__":

    app = QApplication(sys.argv)

    janela = TradutorApp()
    janela.resize(600, 400)
    janela.show()

    sys.exit(app.exec_())


---

3️⃣ Como funciona

1️⃣ Selecionar pasta origem

Contém ficheiros:

texto1.txt
texto2.txt
texto3.txt

2️⃣ Selecionar pasta destino

O programa cria:

texto1.md
texto2.md
texto3.md


---

4️⃣ Exemplo conversão Markdown

TXT original:

Python Programming

Python is a powerful language.
It is used for AI.

Markdown criado:

# Programação Python

Python é uma linguagem poderosa.
É usada para IA.


---

5️⃣ Melhorias possíveis

Pode melhorar muito este programa:

✔ barra de progresso
✔ botão parar tradução
✔ suportar PDF / DOCX / HTML
✔ manter formatação automática Markdown
✔ traduzir README automaticamente para GitHub
✔ criar modo lote para GitHub repos
