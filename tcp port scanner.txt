import socket
import threading
import logging

# Logging setup
logging.basicConfig(filename="scan_results.log", level=logging.INFO,
                    format="%(asctime)s - %(message)s")

# Function to scan a single port
def scan_port(host, port):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(1)  # timeout 1 second
        result = s.connect_ex((host, port))
        if result == 0:
            status = "OPEN"
        else:
            status = "CLOSED"
        print(f"Port {port}: {status}")
        logging.info(f"Port {port}: {status}")
        s.close()
    except socket.timeout:
        print(f"Port {port}: TIMEOUT")
        logging.info(f"Port {port}: TIMEOUT")
    except Exception as e:
        print(f"Port {port}: ERROR ({e})")
        logging.error(f"Port {port}: ERROR ({e})")

# Function to scan a range of ports using threads
def scan_range(host, start_port, end_port):
    threads = []
    for port in range(start_port, end_port + 1):
        t = threading.Thread(target=scan_port, args=(host, port))
        threads.append(t)
        t.start()
    for t in threads:
        t.join()

# Main program
if __name__ == "__main__":
    host = input("Enter host (e.g. scanme.nmap.org): ")
    choice = input("Scan single port (s) or range (r)? ")

    if choice.lower() == "s":
        port = int(input("Enter port: "))
        scan_port(host, port)
    else:
        start = int(input("Start port: "))
        end = int(input("End port: "))
        scan_range(host, start, end)

    print("\n✅ Scan complete. Results saved in scan_results.log")
