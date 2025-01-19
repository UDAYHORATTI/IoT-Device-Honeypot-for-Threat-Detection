# IoT-Device-Honeypot-for-Threat-Detection
This project creates a honeypot system to attract and log malicious activity targeting IoT devices. The data can be used to study attack patterns and improve security measures.
import socket
import threading
import logging
from datetime import datetime

# Configuration
HONEYPOT_IP = "0.0.0.0"  # Listen on all interfaces
HONEYPOT_PORT = 8080     # Common port for IoT devices
LOG_FILE = "honeypot_logs.txt"

# Set up logging
logging.basicConfig(filename=LOG_FILE, level=logging.INFO, format='%(asctime)s - %(message)s')

def log_attack(client_ip, client_port, data):
    """
    Log attack details to a file.
    """
    logging.info(f"Attack from {client_ip}:{client_port} - Data: {data}")
    print(f"[ALERT] Attack logged from {client_ip}:{client_port}")

def handle_connection(client_socket, client_address):
    """
    Handle interaction with a connected client (potential attacker).
    """
    client_ip, client_port = client_address
    print(f"[INFO] Connection from {client_ip}:{client_port}")

    try:
        # Send a fake banner (e.g., mimicking an IoT device)
        banner = "IoT Device v1.0\r\n"
        client_socket.send(banner.encode())

        # Receive data from the client
        data = client_socket.recv(1024).decode()
        log_attack(client_ip, client_port, data)

        # Respond with a fake acknowledgment
        response = "OK\r\n"
        client_socket.send(response.encode())
    except Exception as e:
        print(f"[ERROR] Error handling connection: {e}")
    finally:
        client_socket.close()

def start_honeypot():
    """
    Start the honeypot server to listen for incoming connections.
    """
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind((HONEYPOT_IP, HONEYPOT_PORT))
    server.listen(5)
    print(f"[INFO] Honeypot listening on {HONEYPOT_IP}:{HONEYPOT_PORT}")

    while True:
        client_socket, client_address = server.accept()
        client_thread = threading.Thread(target=handle_connection, args=(client_socket, client_address))
        client_thread.start()

if __name__ == "__main__":
    try:
        start_honeypot()
    except KeyboardInterrupt:
        print("\n[INFO] Honeypot shutting down.")
