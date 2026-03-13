### tradutor completo

✅ Seleciona pasta de origem
✅ Seleciona pasta de destino
✅ Traduz todos os ficheiros de texto
✅ Converte para Markdown (.md)
✅ Permite escolher língua origem e destino (todas)
✅ Botões Traduzir / Parar / Sair
✅ Mostra log na janela

Vou usar:

PyQt5 → interface gráfica

deep-translator → tradução automática

thread → para não bloquear a interface



---

1️⃣ Instalar bibliotecas

pip install PyQt5 deep-translator


---

2️⃣ Script completo

# ==========================================================
# Tradutor de ficheiros para Markdown
# Interface gráfica em Qt
# ==========================================================

import sys
import os
from PyQt5.QtWidgets import *
from PyQt5.QtCore import QThread, pyqtSignal
from deep_translator import GoogleTranslator

# ==========================================================
# THREAD DE TRADUÇÃO
# ==========================================================

class TradutorThread(QThread):

    log = pyqtSignal(str)
    terminado = pyqtSignal()

    def __init__(self, origem, destino, lang_origem, lang_destino):
        super().__init__()

        self.origem = origem
        self.destino = destino
        self.lang_origem = lang_origem
        self.lang_destino = lang_destino

        self.ativo = True


    def run(self):

        self.log.emit("Iniciar tradução...")

        for root, dirs, files in os.walk(self.origem):

            for file in files:

                if not self.ativo:
                    self.log.emit("Tradução parada")
                    self.terminado.emit()
                    return

                if file.endswith(".txt") or file.endswith(".md"):

                    caminho = os.path.join(root, file)

                    try:

                        with open(caminho, "r", encoding="utf-8") as f:
                            texto = f.read()

                        tradutor = GoogleTranslator(
                            source=self.lang_origem,
                            target=self.lang_destino
                        )

                        traduzido = tradutor.translate(texto)

                        nome_saida = os.path.splitext(file)[0] + ".md"
                        destino_file = os.path.join(self.destino, nome_saida)

                        with open(destino_file, "w", encoding="utf-8") as f:
                            f.write(traduzido)

                        self.log.emit(f"Traduzido: {file}")

                    except Exception as e:

                        self.log.emit(f"Erro: {file} -> {str(e)}")

        self.log.emit("Tradução terminada")
        self.terminado.emit()


    def parar(self):
        self.ativo = False


# ==========================================================
# JANELA PRINCIPAL
# ==========================================================

class Janela(QMainWindow):

    def __init__(self):

        super().__init__()

        self.setWindowTitle("Tradutor Markdown Python")
        self.resize(700,500)

        layout = QVBoxLayout()

        # pasta origem
        self.origem = QLineEdit()
        bot_origem = QPushButton("Selecionar Origem")

        bot_origem.clicked.connect(self.selecionar_origem)

        # pasta destino
        self.destino = QLineEdit()
        bot_destino = QPushButton("Selecionar Destino")

        bot_destino.clicked.connect(self.selecionar_destino)

        # linguas
        self.lang_origem = QComboBox()
        self.lang_destino = QComboBox()

        linguas = [
            "auto","en","pt","es","fr","de","it","ru",
            "zh-CN","ja","ko","ar"
        ]

        self.lang_origem.addItems(linguas)
        self.lang_destino.addItems(linguas)

        # botões
        self.bot_traduzir = QPushButton("Traduzir")
        self.bot_parar = QPushButton("Parar")
        self.bot_sair = QPushButton("Sair")

        self.bot_traduzir.clicked.connect(self.traduzir)
        self.bot_parar.clicked.connect(self.parar)
        self.bot_sair.clicked.connect(self.close)

        # log
        self.log = QTextEdit()
        self.log.setReadOnly(True)

        # layout
        layout.addWidget(QLabel("Pasta Origem"))
        layout.addWidget(self.origem)
        layout.addWidget(bot_origem)

        layout.addWidget(QLabel("Pasta Destino"))
        layout.addWidget(self.destino)
        layout.addWidget(bot_destino)

        layout.addWidget(QLabel("Língua Origem"))
        layout.addWidget(self.lang_origem)

        layout.addWidget(QLabel("Língua Destino"))
        layout.addWidget(self.lang_destino)

        layout.addWidget(self.bot_traduzir)
        layout.addWidget(self.bot_parar)
        layout.addWidget(self.bot_sair)

        layout.addWidget(QLabel("Log"))
        layout.addWidget(self.log)

        widget = QWidget()
        widget.setLayout(layout)

        self.setCentralWidget(widget)

        self.thread = None


    # selecionar pasta origem
    def selecionar_origem(self):

        pasta = QFileDialog.getExistingDirectory(self,"Selecionar pasta")

        if pasta:
            self.origem.setText(pasta)


    # selecionar pasta destino
    def selecionar_destino(self):

        pasta = QFileDialog.getExistingDirectory(self,"Selecionar pasta")

        if pasta:
            self.destino.setText(pasta)


    # iniciar tradução
    def traduzir(self):

        origem = self.origem.text()
        destino = self.destino.text()

        lang_o = self.lang_origem.currentText()
        lang_d = self.lang_destino.currentText()

        if origem == "" or destino == "":
            QMessageBox.warning(self,"Erro","Selecionar pastas")
            return

        self.thread = TradutorThread(origem,destino,lang_o,lang_d)

        self.thread.log.connect(self.mostrar_log)

        self.thread.start()


    # parar
    def parar(self):

        if self.thread:
            self.thread.parar()


    # log
    def mostrar_log(self,txt):

        self.log.append(txt)


# ==========================================================
# MAIN
# ==========================================================

if __name__ == "__main__":

    app = QApplication(sys.argv)

    janela = Janela()
    janela.show()

    sys.exit(app.exec_())


---

3️⃣ Como funciona

Interface

A janela tem:

Pasta origem
Pasta destino
Língua origem
Língua destino

[Traduzir]
[Parar]
[Sair]

Log


---

Processo

1️⃣ Selecionar pasta com ficheiros

txt
md

2️⃣ Selecionar pasta destino

3️⃣ Escolher língua

Exemplo

origem: en
destino: pt

4️⃣ Carregar Traduzir


---

Resultado

Exemplo:

docs/
   readme.txt
   tutorial.txt

Destino:

saida/
   readme.md
   tutorial.md


---

4️⃣ Exemplo de conversão

Entrada

Hello world
This is a test

Saída .md

Olá mundo
Isto é um teste


---

5️⃣ Melhorias que posso adicionar

Posso fazer uma versão muito mais avançada com:

✅ suporte PDF → Markdown
✅ suporte DOCX → Markdown
✅ tradução offline com IA
✅ preservação de estrutura Markdown
✅ barra de progresso
✅ menu Qt profissional
✅ log XML / JSON
✅ multithread para traduzir milhares de ficheiros
✅ detecção automática da língua

Posso também transformar isto num projeto profissional para GitHub com:

Python-Translator-MD
│
├── main.py
├── translator.py
├── gui.py
├── requirements.txt
└── README.md

