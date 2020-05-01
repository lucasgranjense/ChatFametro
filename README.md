# ChatFametro
import socket
import sys
from thread import *
import time

x = time.localtime()

""" O primeiro argumento AF_INET é o domínio da direção do soquete. 
Ele é usado para armazenar um domínio de internet com hosts. 
O segundo argumento é o tipo de soquete. SOCK_STREAM significa que os dados 
são exibidos em um continuo gráfico"""
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

# Vamos a verificar si tenemos los argumentos necesarios
if len(sys.argv) != 3:
    print
    "Error: Ao introduzir IP e PORTA no server.py "
    exit()

# O primeiro argumento que recebermos será o ip
IP_address = str(sys.argv[1])

# O segundo para a porta
Port = int(sys.argv[2])

"""Vincular o servidor a um ip e uma porta"""
server.bind((IP_address, Port))

"""Sem limites de conexões"""
server.listen()

# Aquí salvar os clientes que se conectam
list_of_clients = []


def clientthread(conn, addr):
    #Um novo cliente conectado
    conn.send("Bem vindo ao Chat {0}!!!".format(nicks[conn]))

    while True:
        try:
            message = conn.recv(2048)
            if message:

                # Imprime direção e mensagem do usuário
                print
                "<" + addr[0] + addr[1] + " " + nicks[conn] + " > " + message + ('{} : {}'.format(str(x[3]), (str(x[4]))))

                #chamamos à função broadcast para enviar mensagem para todos
                message_to_send = "<" + nicks[conn] + "> " + message
                broadcast(message_to_send, conn)
            else:
                """A mensagem não contém conteúdo e conexão
                está rodando, neste caso eliminamos a conexão """

                remove(conn)


        except:
            continue


"""Usando a função abaixo, transmite a mensagem para todos os clientes"""


def broadcast(message, connection):
    for clients in list_of_clients:
        if clients != connection:
            try:
                clients.send(message)
            except:
                clients.close()

            #Se o laço estiver quebrado, excluiremos o cliente
            remove(clients)


"""A função a seguir simplesmente remove o objeto da lista criada no início """


def remove(connection):
    if connection in list_of_clients:
        list_of_clients.remove(connection)


while True:
    """Aceita uma solicitação de conexão e armazena dois parâmetros, com um objeto de soquete
    e endereço contendo o endereço IP do cliente"""
    conn, addr = server.accept()
    #A primeira mensagem que recebermos será o nick, vamos vê-lo no cliente quando o servidor terminar
    nick = conn.recv(2048)

    """Colocamos o cliente na lista de clientes"""
    list_of_clients.append(conn)

    """Verificamos que o nick não está no nosso biblioteca de nick, para que não seja repetido"""
    t = True
    for i in nicks:
        if nicks[i] == nick:
            t = True
        else:
            t = False
    #Se disponível, adicionamos
    if t == False:
        nicks[conn] = nick

        # Enviamos uma confirmação ao cliente
        conn.send("1")
    else:

        # Se não estiver disponível, enviamos uma negação ao servidor e criamos um loop até que esteja disponível
        while t:
            conn.send("0")
            nick = conn.recv(2048)
            if nicks[i] == nick:
                t = True
            else:
                t = False
                nicks[conn] = nick
                conn.send("1")
    #Agora, imprimimos o endereço e o apelido do usuário que acabou de conectar
    print
    addr[0] + addr[1] " " + nick + " conectado"

    #Criamos um processo individual para o cliente de conexão
    start_new_thread(clientthread, (conn, addr))
conn.close()
server.close()
