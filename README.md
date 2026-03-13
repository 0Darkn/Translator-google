
Esta versão:

✔ interface PyQt5
✔ usa Google Translate via googletrans
✔ traduz todos os ficheiros .txt
✔ gera Markdown válido (.md)
✔ mantém parágrafos corretos
✔ cria títulos automáticos
✔ tem log na janela


---

Script completo — Tradutor TXT → Markdown

# ==========================================================
# TRADUTOR DE FICHEIROS PARA MARKDOWN
# Interface Qt + Google Translate
#
# Funções:
#  - selecionar pasta origem
#  - selecionar pasta destino
#  - escolher línguas
#  - traduzir ficheiros .txt
#  - converter para Markdown
# ==========================================================

import sys
import os

from PyQt5.QtWidgets import (
    QApplication, QWidget, QPushButton, QLabel,
    QFileDialog, QTextEdit, QComboBox,
    QVBoxLayout, QHBoxLayout
)

from googletrans import Translator, LANGUAGES


# ==========================================================
# Classe principal
# ==========================================================

class TradutorQt(QWidget):

    def __init__(self):

        super().__init__()

        self.pasta_origem = ""
        self.pasta_destino = ""

        self.translator = Translator()

        self.criar_interface()


# ==========================================================
# Interface gráfica
# ==========================================================

    def criar_interface(self):

        self.setWindowTitle("Tradutor TXT → Markdown")

        layout_principal = QVBoxLayout()

        # -------------------------
        # Língua origem
        # -------------------------

        layout_principal.addWidget(QLabel("Língua origem"))

        self.combo_origem = QComboBox()

        # -------------------------
        # Língua destino
        # -------------------------

        layout_principal.addWidget(QLabel("Língua destino"))

        self.combo_destino = QComboBox()

        # adicionar todas as línguas

        for codigo, nome in LANGUAGES.items():

            self.combo_origem.addItem(nome, codigo)
            self.combo_destino.addItem(nome, codigo)

        self.combo_origem.setCurrentText("english")
        self.combo_destino.setCurrentText("portuguese")

        layout_principal.addWidget(self.combo_origem)
        layout_principal.addWidget(self.combo_destino)

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

        layout_principal.addLayout(botoes)

        # -------------------------
        # Log
        # -------------------------

        self.log = QTextEdit()
        self.log.setReadOnly(True)

        layout_principal.addWidget(self.log)

        self.setLayout(layout_principal)

        # -------------------------
        # Ligações
        # -------------------------

        self.btn_origem.clicked.connect(self.escolher_origem)
        self.btn_destino.clicked.connect(self.escolher_destino)
        self.btn_traduzir.clicked.connect(self.traduzir_ficheiros)
        self.btn_sair.clicked.connect(self.close)


# ==========================================================
# Escolher pasta origem
# ==========================================================

    def escolher_origem(self):

        pasta = QFileDialog.getExistingDirectory(
            self,
            "Selecionar pasta origem"
        )

        if pasta:

            self.pasta_origem = pasta
            self.log.append(f"Pasta origem: {pasta}")


# ==========================================================
# Escolher pasta destino
# ==========================================================

    def escolher_destino(self):

        pasta = QFileDialog.getExistingDirectory(
            self,
            "Selecionar pasta destino"
        )

        if pasta:

            self.pasta_destino = pasta
            self.log.append(f"Pasta destino: {pasta}")


# ==========================================================
# Converter texto para Markdown
# ==========================================================

    def converter_markdown(self, texto):

        linhas = texto.split("\n")

        md = []

        for i, linha in enumerate(linhas):

            linha = linha.strip()

            if linha == "":
                md.append("")
                continue

            # primeira linha vira título
            if i == 0:

                md.append("# " + linha)

            else:

                md.append(linha)

        return "\n".join(md)


# ==========================================================
# Traduzir ficheiros
# ==========================================================

    def traduzir_ficheiros(self):

        if self.pasta_origem == "" or self.pasta_destino == "":

            self.log.append("Selecione as pastas primeiro.")
            return

        idioma_origem = self.combo_origem.currentData()
        idioma_destino = self.combo_destino.currentData()

        ficheiros = os.listdir(self.pasta_origem)

        for nome in ficheiros:

            if not nome.endswith(".txt"):
                continue

            caminho = os.path.join(self.pasta_origem, nome)

            self.log.append(f"Traduzindo: {nome}")

            try:

                with open(caminho, "r", encoding="utf-8") as f:
                    texto = f.read()

                traduzido = self.translator.translate(
                    texto,
                    src=idioma_origem,
                    dest=idioma_destino
                ).text

                md = self.converter_markdown(traduzido)

                novo_nome = nome.replace(".txt", ".md")

                destino = os.path.join(
                    self.pasta_destino,
                    novo_nome
                )

                with open(destino, "w", encoding="utf-8") as f:
                    f.write(md)

                self.log.append(f"Gravado: {novo_nome}")

            except Exception as erro:

                self.log.append(f"Erro: {erro}")

        self.log.append("Tradução terminada.")


# ==========================================================
# Executar programa
# ==========================================================

if __name__ == "__main__":

    app = QApplication(sys.argv)

    janela = TradutorQt()

    janela.resize(700, 500)

    janela.show()

    sys.exit(app.exec_())


---

Como executar

1️⃣ instalar bibliotecas

pip install PyQt5
pip install googletrans==4.0.0-rc1

2️⃣ correr

python tradutor_qt_md.py


---

Resultado

Entrada:

hello.txt

conteúdo:

Python Programming

Python is easy to learn.
It is used in AI.

Saída:

hello.md

# Programação Python

Python é fácil de aprender.
É usado em IA.



---
