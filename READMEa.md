
Python Qt File Translator → Markdown

Tradutor de ficheiros feito em Python + Qt, que permite traduzir ficheiros de uma pasta para outra e converter automaticamente para Markdown (.md).

A aplicação possui uma interface gráfica, seleção de pastas e suporte para múltiplas línguas.


---

Funcionalidades

Selecionar pasta de origem

Selecionar pasta de destino

Traduzir ficheiros .txt e .md

Converter automaticamente para Markdown

Suporte para várias línguas

Interface gráfica com Qt

Botões:


Traduzir
Parar
Sair

Área de log

Tradução executada em thread para não bloquear a interface



---

Tecnologias usadas

Python

PyQt5

deep-translator



---

Instalação

Instalar as bibliotecas necessárias:

pip install PyQt5 deep-translator


---

Script completo

import sys
import os
from PyQt5.QtWidgets import *
from PyQt5.QtCore import QThread, pyqtSignal
from deep_translator import GoogleTranslator


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


class Janela(QMainWindow):

    def __init__(self):

        super().__init__()

        self.setWindowTitle("Tradutor Markdown Python")
        self.resize(700,500)

        layout = QVBoxLayout()

        self.origem = QLineEdit()
        bot_origem = QPushButton("Selecionar Origem")
        bot_origem.clicked.connect(self.selecionar_origem)

        self.destino = QLineEdit()
        bot_destino = QPushButton("Selecionar Destino")
        bot_destino.clicked.connect(self.selecionar_destino)

        self.lang_origem = QComboBox()
        self.lang_destino = QComboBox()

        linguas = [
            "auto","en","pt","es","fr","de","it",
            "ru","zh-CN","ja","ko","ar"
        ]

        self.lang_origem.addItems(linguas)
        self.lang_destino.addItems(linguas)

        self.bot_traduzir = QPushButton("Traduzir")
        self.bot_parar = QPushButton("Parar")
        self.bot_sair = QPushButton("Sair")

        self.bot_traduzir.clicked.connect(self.traduzir)
        self.bot_parar.clicked.connect(self.parar)
        self.bot_sair.clicked.connect(self.close)

        self.log = QTextEdit()
        self.log.setReadOnly(True)

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


    def selecionar_origem(self):

        pasta = QFileDialog.getExistingDirectory(self,"Selecionar pasta")

        if pasta:
            self.origem.setText(pasta)


    def selecionar_destino(self):

        pasta = QFileDialog.getExistingDirectory(self,"Selecionar pasta")

        if pasta:
            self.destino.setText(pasta)


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


    def parar(self):

        if self.thread:
            self.thread.parar()


    def mostrar_log(self,txt):

        self.log.append(txt)


if __name__ == "__main__":

    app = QApplication(sys.argv)

    janela = Janela()
    janela.show()

    sys.exit(app.exec_())


---

Como usar

1. Executar o script:



python tradutor.py

2. Selecionar:



Pasta origem
Pasta destino

3. Escolher a língua:



origem
destino

4. Carregar em Traduzir




---

Exemplo

Pasta origem

docs/
   file1.txt
   tutorial.txt

Pasta destino

output/
   file1.md
   tutorial.md


---

Exemplo de tradução

Entrada:

Hello world
This is a test

Saída:

Olá mundo
Isto é um teste


---

Melhorias futuras

suporte PDF → Markdown

suporte DOCX → Markdown

barra de progresso

tradução offline

multithread para milhares de ficheiros

logs em XML ou JSON

menu Qt avançado



---
