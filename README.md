# Bibliotecas Utilizadas
from customtkinter import CTkButton, set_appearance_mode, CTkFrame, CTkLabel, CTkEntry, CTkTabview, CTk, \
    set_default_color_theme, CTkCheckBox, CTkOptionMenu
import csv # Pensar em otimização, utilizando SQLite
import datetime
import matplotlib.pyplot as plt
from collections import defaultdict
from tkinter import messagebox, ttk  # Importando ttk para usar o Combobox
from PIL import Image, ImageTk
from cryptography.fernet import Fernet
import os
import re

# Configurações iniciais
set_appearance_mode("light")  # Modo claro
set_default_color_theme("blue")  # Cor de destaque azul

# Arquivos para armazenar dados. Pensar em otimização, utilizando SQLite.
arquivo_usuarios = "usuarios.csv"
arquivo_dados = "gastos.csv"
arquivo_chave = "chave.key"
diretorio_imagens = "./Imagens"  # Diretório de imagens
arquivo_metas = "metas.csv"

# Configuração da chave de criptografia
if not os.path.exists(arquivo_chave):
    chave_criptografia = Fernet.generate_key()
    with open(arquivo_chave, 'wb') as file:
        file.write(chave_criptografia)
else:
    with open(arquivo_chave, 'rb') as file:
        chave_criptografia = file.read()

fernet = Fernet(chave_criptografia)

# Inicialização do arquivo_usuarios
if not os.path.exists(arquivo_usuarios):
    with open(arquivo_usuarios, 'w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(["Nome", "Email", "Login", "Senha", "Telefone"])

# Inicialização do arquivo_dados
if not os.path.exists(arquivo_dados):
    with open(arquivo_dados, 'w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(["Data", "Categoria", "Valor"])

# Inicialização do arquivo_metas
if not os.path.exists(arquivo_metas):
    with open(arquivo_metas, 'w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(["Título", "Meta Mensal"])

# Lista de categorias pré-definidas
categorias_predefinidas = ["Habitação", "Transporte", "Alimentação", "Saúde", "Educação", "Lazer", "Outros"]

# Funções auxiliares
def criptografar_senha(senha):
    return fernet.encrypt(senha.encode()).decode()

def descriptografar_senha(senha_criptografada):
    return fernet.decrypt(senha_criptografada.encode()).decode()

def validar_senha(senha):
    # Senha deve ter ao menos 8 caracteres, uma letra, um número e um caractere especial
    padrao = r'^(?=.*[A-Za-z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$'
    return re.match(padrao, senha) is not None

def autenticar_usuario(login, senha):
    try:
        with open(arquivo_usuarios, 'r', newline='', encoding='utf-8') as file:
            reader = csv.DictReader(file)
            for row in reader:
                if row["Login"] == login and descriptografar_senha(row["Senha"]) == senha:
                    return row
        return None
    except Exception as e:
        messagebox.showerror("Erro", f"Erro ao autenticar: {e}")

def registrar_usuario(nome, email, login, senha, telefone):
    if not validar_email(email):
        messagebox.showerror("Erro", "E-mail inválido.")
        return False
    if telefone and not validar_telefone(telefone):
        messagebox.showerror("Erro", "Telefone inválido.")
        return False
    if not validar_senha(senha):
        messagebox.showerror(
            "Erro",
            "Senha inválida. Deve conter pelo menos 8 caracteres, incluindo uma letra, um número e um caractere especial."
        )
        return False
    try:
        with open(arquivo_usuarios, 'a', newline='', encoding='utf-8') as file:
            writer = csv.writer(file)
            writer.writerow([nome, email, login, criptografar_senha(senha), telefone])
        return True
    except Exception as e:
        messagebox.showerror("Erro", f"Erro ao registrar: {e}")
        return False

def validar_email(email):
    padrao = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(padrao, email) is not None

def validar_telefone(telefone):
    return telefone.isdigit() and (8 <= len(telefone) <= 15)

# Função para carregar e salvar dados em CSV
def carregar_dados():
    gastos = defaultdict(list)
    try:
        with open(arquivo_dados, 'r', newline='', encoding='utf-8') as file:
            leitor_csv = csv.reader(file)
            next(leitor_csv)  # Pular cabeçalho
            for linha in leitor_csv:
                data, categoria, valor = linha
                gastos[categoria].append({'data': data, 'valor': float(valor)})
    except FileNotFoundError:
        pass
    return gastos

def salvar_dados(dados):
    with open(arquivo_dados, 'w', newline='', encoding='utf-8') as file:
        escritor_csv = csv.writer(file)
        escritor_csv.writerow(['Data', 'Categoria', 'Valor'])  # Cabeçalho
        for categoria, entradas in dados.items():
            for entrada in entradas:
                escritor_csv.writerow([entrada['data'], categoria, entrada['valor']])

def listar_usuarios():
    try:
        with open(arquivo_usuarios, 'r', newline='', encoding='utf-8') as file:
            reader = csv.DictReader(file)
            usuarios = [{"Nome": row["Nome"], "Email": row["Email"], "Login": row["Login"]} for row in reader]
        return usuarios
    except Exception as e:
        messagebox.showerror("Erro", f"Erro ao listar usuários: {e}")
        return []

# Inicialização dos dados de gastos
gastos = carregar_dados()

#Função para exibir o resumo de gastos
# Confirmar motivo para não exibir os gastos
def exibir_resumo(frame_resumo_gastos):
    CTkLabel(frame_resumo_gastos, text="Resumo de Gastos", font=("Arial", 18)).pack(side="top")
    label_resumo = CTkLabel(frame_resumo_gastos, text="Escolha a Categoria:")
    label_resumo.pack(pady=10, padx=5)
    escolha_categoria = ttk.Combobox(frame_resumo_gastos, values=categorias_predefinidas)  # Usando o Combobox do ttk
    escolha_categoria.pack(pady=5)

# Funções para criar frames de conteúdo
# Função para criar "Página Inicial"
def criar_pagina_inicial(frame):
    for widget in frame.winfo_children():
        widget.destroy()

    frame_pagina_inicial = CTkFrame(frame )
    frame_pagina_inicial.pack(fill="both")

    # Título da página inicial
    CTkLabel(frame_pagina_inicial, text="Seja bem-vindo ao seu Assistente Pessoal de Finanças", font=("Helvetica", 36)).pack(pady=30)
    # Subtítulo
    CTkLabel(frame_pagina_inicial, text="Escolha uma das abas acima e comece o seu planejamento financeiro para alcançar os seus sonhos", font=("Helvetica", 18)).pack(pady=10)

    # Carregar e exibir imagens de nuvens
    imagens_nuvens = []
    for _ in range(4):
        imagem = Image.open("/Users/guilhermevilalima/Desktop/Assistente Financeiro Pessoal/Imagens/Nuvem.png")  # Altere para o caminho correto da imagem
        imagem = imagem.resize((250, 100))  # Aumentar o tamanho das imagens de nuvens
        imagem_tk = ImageTk.PhotoImage(imagem)
        imagens_nuvens.append(imagem_tk)

    frame_nuvens = CTkFrame(frame)
    frame_nuvens.pack(pady=20, padx=30)  # Ajustar para preencher a largura disponível

    for imagem in imagens_nuvens:
        label_imagem = CTkLabel(frame_nuvens, image=imagem, text="")
        label_imagem.pack(side="left", padx=5)  # Exibir as imagens lado a lado
    frame.imagens_nuvens = imagens_nuvens  # Manter referência para evitar coleta de lixo

    # Criar frame para exibir resumo de gastos
    frame_resumo_gastos = CTkFrame(frame)
    frame_resumo_gastos.pack(pady=10, fill="x")
    exibir_resumo(frame_resumo_gastos)

# Função para criar a aba "Metas"
def criar_aba_metas(frame):
    for widget in frame.winfo_children():
        widget.destroy()

    frame_metas = CTkFrame(frame)
    frame_metas.pack (pady=5, fill="x")
    CTkLabel(frame_metas, text="Defina suas metas financeiras", font=("Arial", 16)).pack(pady=10)

    CTkLabel(frame_metas, text="Meta Mensal (R$):", font=("Arial", 14)).pack(pady=5)
    meta_mensal_entry = CTkEntry(frame_metas, placeholder_text="Insira o valor da meta")
    meta_mensal_entry.pack(pady=5)

    def salvar_meta():
        meta_mensal = meta_mensal_entry.get()
        if meta_mensal.isdigit():
            with open(arquivo_metas, 'w', newline='', encoding='utf-8') as file:
                writer = csv.writer(file)
                writer.writerow(["Meta Mensal"])
                writer.writerow([meta_mensal])
            messagebox.showinfo("Meta", f"Meta de R$ {meta_mensal} salva com sucesso!")
        else:
            messagebox.showerror("Erro", "Por favor, insira um valor numérico.")

    CTkButton(frame_metas, text="Salvar Meta", command=salvar_meta).pack(pady=10)

def adicionar_gasto_frame(frame):
    for widget in frame.winfo_children():
        widget.destroy()

    def confirmar_gasto():
        data = entrada_data.get()
        categoria = entrada_categoria.get()
        valor = entrada_valor.get()

        if categoria not in gastos:
            resposta = messagebox.askyesno("Nova Categoria", "Categoria não encontrada. Deseja criar uma nova?")
            if resposta:
                gastos[categoria] = []
                messagebox.showinfo("Sucesso", f"Categoria '{categoria}' criada com sucesso!")

        try:
            valor_float = float(valor)
            gastos[categoria].append({'data': data, 'valor': valor_float})
            salvar_dados(gastos)
            messagebox.showinfo("Sucesso", "Gasto adicionado com sucesso!")
        except ValueError:
            messagebox.showerror("Erro", "Por favor, insira um valor numérico válido.")

    CTkLabel(frame, text="Data (AAAA-MM-DD):").pack(pady=5)
    entrada_data = CTkEntry(frame)
    entrada_data.pack(pady=5)

    # Criar Combobox com categorias pré-definidas
    CTkLabel(frame, text="Categoria:").pack(pady=5)
    entrada_categoria = ttk.Combobox(frame, values=categorias_predefinidas)  # Usando o Combobox do ttk
    entrada_categoria.pack(pady=5)

    # Campo para digitar uma nova categoria, caso necessário
    CTkLabel(frame, text="Ou insira uma nova categoria:").pack(pady=5)
    entrada_nova_categoria = CTkEntry(frame)
    entrada_nova_categoria.pack(pady=5)

    # Funcionalidade para adicionar nova categoria
    def adicionar_nova_categoria():
        nova_categoria = entrada_nova_categoria.get().strip().capitalize()
        if nova_categoria and nova_categoria not in categorias_predefinidas:
            categorias_predefinidas.append(nova_categoria)
            messagebox.showinfo("Sucesso", f"Categoria '{nova_categoria}' adicionada com sucesso!")
            entrada_nova_categoria.delete(0, 'end')  # Limpar o campo após adicionar
        elif nova_categoria in categorias_predefinidas:
            messagebox.showwarning("Aviso", "Esta categoria já existe.")
        else:
            messagebox.showwarning("Aviso", "Por favor, insira um nome para a categoria.")

    CTkButton(frame, text="Adicionar Nova Categoria", command=adicionar_nova_categoria).pack(pady=5)

    CTkLabel(frame, text="Valor:").pack(pady=5)
    entrada_valor = CTkEntry(frame)
    entrada_valor.pack(pady=5)

    CTkButton(frame, text="adicionar Gasto", command=confirmar_gasto).pack(pady=5)

def exibir_gastos_mes_frame(frame):
    for widget in frame.winfo_children():
        widget.destroy()

    def calcular_gastos():
        ano = entrada_ano.get()
        mes = entrada_mes.get()

        try:
            ano_int = int(ano)
            mes_int = int(mes)

            total_gastos = {}
            for categoria, entradas in gastos.items():
                total_categoria = sum(entry['valor'] for entry in entradas if entry['data'].startswith(f"{ano_int}-{mes_int:02d}"))
                total_gastos[categoria] = total_categoria

            total_geral = sum(total_gastos.values())
            resultado['text'] = f"Total gasto em {ano}-{mes}: R$ {total_geral:.2f}"

            if total_geral > 0:
                plt.figure(figsize=(8, 6))
                plt.pie(total_gastos.values(), labels = total_gastos.keys(), autopct='%1.1f%%')
                plt.title(f"Gastos por Categoria em {ano}-{mes}")
                plt.show()
            else:
                messagebox.showinfo("Info", "Nenhum gasto registrado para esse mês.")
        except ValueError:
            messagebox.showerror("Erro", "Por favor, insira ano e mês válidos.")

    CTkLabel(frame, text="Ano (AAAA):").pack(pady=5)
    entrada_ano = CTkEntry(frame)
    entrada_ano.pack(pady=5)
    CTkLabel(frame, text="Mês (MM):").pack(pady=5)
    entrada_mes = CTkEntry(frame)
    entrada_mes.pack(pady=5)
    resultado = CTkLabel(frame, text="")
    resultado.pack(pady=5)
    CTkButton(frame, text="Calcular", command=calcular_gastos).pack(pady=5)

# Função para alternar configurações visuais
def criar_aba_configuracoes(frame):
    frame_configuracoes = CTkFrame(frame)
    frame_configuracoes.pack(pady=5, fill="x")

    # Criar área para personalizar o visual
    CTkLabel(frame_configuracoes, text="Personalize o Visual", font=("Arial", 16)).pack(pady=10)

    # Criar barra de escolha de temas
    # Dúvida: O tema não está atualizando automaticamente
    CTkLabel(frame_configuracoes, text="Escolha um tema:")
    menu_tema = CTkOptionMenu(frame_configuracoes, values=["blue", "green", "dark-blue"],
                              command=lambda value: set_default_color_theme(value))
    menu_tema.pack(pady=10)

    # criar barra de escolha de modos
    CTkLabel(frame_configuracoes, text="escolha um modo")
    menu_modo = CTkOptionMenu(frame_configuracoes, values=["light", "dark", "system"],
                              command=lambda value: set_appearance_mode(value))
    menu_modo.pack(pady=10)


def tela_cadastro(frame):
    for widget in frame.winfo_children():
        widget.destroy()

    CTkLabel(frame, text="Cadastro de Usuário", font=("Helvetica", 26)).pack(pady=30)

    CTkLabel(frame, text="Nome:").pack(pady=5)
    entrada_nome = CTkEntry(frame)
    entrada_nome.pack(pady=5)

    CTkLabel(frame, text="E-mail:").pack(pady=5)
    entrada_email = CTkEntry(frame)
    entrada_email.pack(pady=5)

    CTkLabel(frame, text="Usuário:").pack(pady=5)
    entrada_usuario = CTkEntry(frame)
    entrada_usuario.pack(pady=5)

    CTkLabel(frame, text="Senha:").pack(pady=5)
    entrada_senha = CTkEntry(frame, show="*")
    entrada_senha.pack(pady=5)

    CTkLabel(frame, text="Telefone (opcional):").pack(pady=5)
    entrada_telefone = CTkEntry(frame)
    entrada_telefone.pack(pady=5)

    # Adicionar opção para não informar telefone
    def telefone_nao_informado():
        entrada_telefone.delete(0, 'end')  # Limpar campo de telefone
        entrada_telefone.config(state="disabled")  # Desabilitar campo

    CTkButton(frame, text="Não informar o telefone", command=telefone_nao_informado).pack(pady=5)

    def cadastrar_usuario():
        nome = entrada_nome.get()
        email = entrada_email.get()
        usuario = entrada_usuario.get()
        senha = entrada_senha.get()
        telefone = entrada_telefone.get() or ""  # Se o telefone não for informado, vai ser uma string vazia

        if nome and email and usuario and senha:
            if cadastrar_usuario:
                messagebox.showinfo("Sucesso", "Cadastro realizado com sucesso!")
                # Limpar campos após o cadastro
                entrada_nome.delete(0, 'end')
                entrada_email.delete(0, 'end')
                entrada_usuario.delete(0, 'end')
                entrada_senha.delete(0, 'end')
                entrada_telefone.delete(0, 'end')
            else:
                messagebox.showerror("Erro", "Erro ao registrar usuário.")
        else:
            messagebox.showwarning("Aviso", "Todos os campos obrigatórios (Nome, E-mail, Usuário e Senha) devem ser preenchidos.")

    CTkButton(frame, text="Cadastrar", command=cadastrar_usuario).pack(pady=20)

def tela_login():
    # Frame de login
    login_frame = CTkFrame(root)
    login_frame.pack(expand=True, fill="both")

    CTkLabel(login_frame, text="Login", font=("Helvetica", 32)).pack(pady=50)

    CTkLabel(login_frame, text="Usuário:").pack(pady=5)
    entrada_usuario = CTkEntry(login_frame)
    entrada_usuario.pack(pady=5)

    CTkLabel(login_frame, text="Senha:").pack(pady=5)
    entrada_senha = CTkEntry(login_frame, show="*")
    entrada_senha.pack(pady=5)

    def login():
        login = entrada_usuario.get()
        senha = entrada_senha.get()

        usuario = autenticar_usuario(login, senha)
        if usuario:
            login_frame.destroy()  # Fechar tela de login
            tabview.pack(expand=True, fill="both")  # Exibir a tela principal

            # Carregar o conteúdo de cada aba
            criar_pagina_inicial(aba_pagina_inicial)
            adicionar_gasto_frame(aba_adicionar_gasto)
            exibir_gastos_mes_frame(aba_exibir_gastos)
            criar_aba_metas(aba_metas)
            criar_aba_configuracoes(aba_configuracoes)

        else:
            messagebox.showerror("Erro", "Login ou senha incorretos.")

    CTkButton(login_frame, text="Entrar", command=login).pack(pady=20)

    def registrar_usuario_popup():
        # Ação para exibir o formulário de cadastro
        for widget in login_frame.winfo_children():
            widget.destroy()  # Limpa os widgets da tela de login

        CTkLabel(login_frame, text="Cadastro", font=("Helvetica", 32)).pack(pady=30)

        # Asterisco e aviso
        CTkLabel(login_frame, text="* Campos de preenchimento obrigatório.", font=("Helvetica", 10), text_color="red").pack(pady=5)

        # Campos de cadastro
        def formatar_nome(nome):
            return " ".join([parte.capitalize() for parte in nome.strip().split()])

        CTkLabel(login_frame, text="* Nome:", text_color="red").pack(pady=5)
        entrada_nome = CTkEntry(login_frame)
        entrada_nome.pack(pady=5)

        CTkLabel(login_frame, text="* E-mail:", text_color="red").pack(pady=5)
        entrada_email = CTkEntry(login_frame)
        entrada_email.pack(pady=5)

        CTkLabel(login_frame, text="Telefone:").pack(pady=5)
        entrada_telefone = CTkEntry(login_frame)
        entrada_telefone.pack(pady=5)

        CTkLabel(login_frame, text="* Usuário:", text_color="red").pack(pady=5)
        entrada_usuario_reg = CTkEntry(login_frame)
        entrada_usuario_reg.pack(pady=5)

        CTkLabel(login_frame, text="* Senha:", text_color="red").pack(pady=5)
        entrada_senha_reg = CTkEntry(login_frame, show="*")
        entrada_senha_reg.pack(pady=5)

        def cadastrar():
            nome = formatar_nome(entrada_nome.get())
            email = entrada_email.get().strip()
            telefone = entrada_telefone.get().strip() or ""  # Permite que o telefone seja vazio
            usuario_reg = entrada_usuario_reg.get().strip()
            senha_reg = entrada_senha_reg.get().strip()

            if not nome or not email or not usuario_reg or not senha_reg:
                messagebox.showerror("Erro", "Por favor, preencha todos os campos obrigatórios.")
                return

            if registrar_usuario(nome, email, usuario_reg, senha_reg, telefone):
                messagebox.showinfo("Sucesso", "Cadastro realizado com sucesso!")
                login_frame.destroy()
                tela_login()  # Volta automaticamente para a tela de login
            else:
                messagebox.showerror("Erro", "Erro ao realizar cadastro.")

        CTkButton(login_frame, text="Cadastrar", command=cadastrar).pack(pady=20)

    CTkButton(login_frame, text="Primeiro Acesso? Cadastre-se aqui.", command=registrar_usuario_popup).pack(pady=20)

# Execução do programa
if __name__ == "__main__":
    root = CTk()
    root.title("Assistente de Finanças Domésticas")
    root.geometry("1200x900")

    tabview = CTkTabview(root)
    aba_pagina_inicial = tabview.add("Página Inicial")
    aba_adicionar_gasto = tabview.add("Adicionar Gasto")
    aba_exibir_gastos = tabview.add("Exibir Relatório")
    aba_metas = tabview.add("Metas")
    aba_configuracoes = tabview.add("Configurações")

    tela_login()  # Chama a tela de login primeiro

    root.mainloop()
