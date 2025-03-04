# socket-marioti
import socket
import sys
import threading

def read_config():
    with open("config.txt", "r") as file:
        host, port = file.read().splitlines()
    return host, int(port)

def chat_server():
    host, port = read_config()
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(1)
    print(f"Servidor ouvindo em {host}:{port}...")
    
    conn, addr = server_socket.accept()
    print(f"Conexão estabelecida com {addr}")
    
    def receive_messages():
        while True:
            msg = conn.recv(1024).decode()
            if not msg:
                break
            print(f"Cliente: {msg}")
    
    threading.Thread(target=receive_messages, daemon=True).start()
    
    while True:
        conn.send(input("Servidor: ").encode())

def chat_client():
    host, port = read_config()
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.connect((host, port))
    print(f"Conectado ao servidor {host}:{port}")
    
    def receive_messages():
        while True:
            msg = client_socket.recv(1024).decode()
            if not msg:
                break
            print(f"Servidor: {msg}")
    
    threading.Thread(target=receive_messages, daemon=True).start()
    
    while True:
        client_socket.send(input("Cliente: ").encode())

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Uso: python script.py server|client")
        sys.exit(1)
    
    role = sys.argv[1].lower()
    if role == "server":
        chat_server()
    elif role == "client":
        chat_client()
    else:
        print("Opção inválida! Use 'server' ou 'client'")
