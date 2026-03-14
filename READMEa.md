Para a tradução, utilizaremos a biblioteca googletrans, que é uma interface gratuita para a API do Google Tradutor.
Aqui está o código completo, estruturado e comentado para você publicar:
Requisitos Práticos
Antes de rodar, instale as bibliotecas necessárias:
pip install PyQt6 googletrans==4.0.0-rc1

Código do Tradutor Multilingue (main.py)
import os
import sys
from PyQt6.QtWidgets import (QApplication, QWidget, QVBoxLayout, QHBoxLayout, 
                             QPushButton, QFileDialog, QLabel, QProgressBar, 
                             QComboBox, QTextEdit, QMessageBox)
from PyQt6.QtCore import Qt, QThread, pyqtSignal
from googletrans import Translator, LANGUAGES

# Classe Worker para processar a tradução em segundo plano (evita travar a interface)
class TranslationWorker(QThread):
    progress = pyqtSignal(int)
    finished = pyqtSignal(str)
    
    def __init__(self, source_dir, dest_dir, target_lang):
        super().__init__()
        self.source_dir = source_dir
        self.dest_dir = dest_dir
        self.target_lang = target_lang
        self.is_running = True
        self.translator = Translator()

    def run(self):
        try:
            files = [f for f in os.listdir(self.source_dir) if f.endswith('.txt')]
            if not files:
                self.finished.emit("Nenhum arquivo .txt encontrado.")
                return

            for i, filename in enumerate(files):
                if not self.is_running:
                    break
                
                # Lendo o arquivo original
                with open(os.path.join(self.source_dir, filename), 'r', encoding='utf-8') as f:
                    content = f.read()

                # Traduzindo
                translated = self.translator.translate(content, dest=self.target_lang)
                
                # Salvando o novo arquivo
                new_path = os.path.join(self.dest_dir, f"translated_{filename}")
                with open(new_path, 'w', encoding='utf-8') as f:
                    f.write(translated.text)

                # Atualizando progresso
                percent = int(((i + 1) / len(files)) * 100)
                self.progress.emit(percent)

            self.finished.emit("Processo concluído com sucesso!")
        except Exception as e:
            self.finished.emit(f"Erro: {str(e)}")

    def stop(self):
        self.is_running = False

# Janela Principal
class TranslatorApp(QWidget):
    def __init__(self):
        super().__init__()
        self.initUI()
        
    def initUI(self):
        self.setWindowTitle('AutoTrans QT - Tradutor de Pastas')
        self.setGeometry(300, 300, 500, 400)
        
        layout = QVBoxLayout()

        # Seleção de Idioma
        self.lang_label = QLabel("Selecione o idioma de destino:")
        self.lang_combo = QComboBox()
        for lang_code, lang_name in LANGUAGES.items():
            self.lang_combo.addItem(lang_name.capitalize(), lang_code)
        self.lang_combo.setCurrentText("Portuguese")
        
        # Botões de Pasta
        self.btn_source = QPushButton("Selecionar Pasta de Origem")
        self.btn_source.clicked.connect(self.select_source)
        self.lbl_source = QLabel("Origem: Não selecionada")
        
        self.btn_dest = QPushButton("Selecionar Pasta de Destino")
        self.btn_dest.clicked.connect(self.select_dest)
        self.lbl_dest = QLabel("Destino: Não selecionado")

        # Controles de Execução
        btn_layout = QHBoxLayout()
        self.btn_start = QPushButton("Traduzir")
        self.btn_start.clicked.connect(self.start_translation)
        
        self.btn_stop = QPushButton("Parar")
        self.btn_stop.setEnabled(False)
        self.btn_stop.clicked.connect(self.stop_translation)
        
        self.btn_exit = QPushButton("Sair")
        self.btn_exit.clicked.connect(self.close)
        
        btn_layout.addWidget(self.btn_start)
        btn_layout.addWidget(self.btn_stop)
        btn_layout.addWidget(self.btn_exit)

        # Progresso
        self.pbar = QProgressBar()
        self.pbar.setValue(0)

        # Adicionando ao Layout
        layout.addWidget(self.lang_label)
        layout.addWidget(self.lang_combo)
        layout.addWidget(self.btn_source)
        layout.addWidget(self.lbl_source)
        layout.addWidget(self.btn_dest)
        layout.addWidget(self.lbl_dest)
        layout.addLayout(btn_layout)
        layout.addWidget(self.pbar)

        self.setLayout(layout)
        self.source_path = ""
        self.dest_path = ""

    def select_source(self):
        path = QFileDialog.getExistingDirectory(self, "Selecionar Pasta de Origem")
        if path:
            self.source_path = path
            self.lbl_source.setText(f"Origem: {path}")

    def select_dest(self):
        path = QFileDialog.getExistingDirectory(self, "Selecionar Pasta de Destino")
        if path:
            self.dest_path = path
            self.lbl_dest.setText(f"Destino: {path}")

    def start_translation(self):
        if not self.source_path or not self.dest_path:
            QMessageBox.warning(self, "Erro", "Selecione as pastas primeiro!")
            return

        target_lang = self.lang_combo.currentData()
        self.worker = TranslationWorker(self.source_path, self.dest_path, target_lang)
        self.worker.progress.connect(self.update_progress)
        self.worker.finished.connect(self.translation_finished)
        
        self.btn_start.setEnabled(False)
        self.btn_stop.setEnabled(True)
        self.worker.start()

    def stop_translation(self):
        if self.worker:
            self.worker.stop()
            self.btn_stop.setEnabled(False)

    def update_progress(self, val):
        self.pbar.setValue(val)

    def translation_finished(self, msg):
        QMessageBox.information(self, "Status", msg)
        self.btn_start.setEnabled(True)
        self.btn_stop.setEnabled(False)
        self.pbar.setValue(0)

if __name__ == '__main__':
    app = QApplication(sys.argv)
    ex = TranslatorApp()
    ex.show()
    sys.exit(app.exec())

O que este código faz:
 * Multi-threading (QThread): Essencial para que a janela não trave (fique "Não Respondendo") enquanto o Google processa os textos.
 * Sinais (pyqtSignal): Comunica o progresso da tradução entre a lógica de tradução e a interface visual.
 * Seleção de Pastas: Usa o QFileDialog para facilitar a navegação no Windows/Linux/Mac.
 * Barra de Progresso: Calcula em tempo real a porcentagem baseada no número de arquivos .txt na pasta.
 * Multilingue: Carrega dinamicamente todos os idiomas suportados pelo Google através do dicionário LANGUAGES.
Dicas para o seu Repositório:
 * README.md: Adicione prints da tela.
 * Encapsulamento: Se quiser transformar em .exe, use o pyinstaller --noconsole --onefile main.py.
 * Licença: Adicione uma licença MIT para ser amigável à comunidade.

