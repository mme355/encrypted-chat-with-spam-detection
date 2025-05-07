import socket
import threading
import rsa
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB

# ------------------ Spam Detection Module ------------------

class SpamClassifier:
    def __init__(self):
        self.vectorizer = CountVectorizer()
        self.classifier = MultinomialNB()
        self._train()

    def _train(self):
        # Sample dataset (can be expanded)
        texts = [
            "win money now", "buy cheap meds", "click here", "hello", "how are you", "let's meet",
            "free offer", "win big", "hi", "are you coming today"
        ]
        labels = [1, 1, 1, 0, 0, 0, 1, 1, 0, 0]  # 1: spam, 0: not spam
        vectors = self.vectorizer.fit_transform(texts)
        self.classifier.fit(vectors, labels)

    def is_spam(self, message):
        vector = self.vectorizer.transform([message])
        prediction = self.classifier.predict(vector)
        return prediction[0] == 1

# Initialize spam detector
spam_detector = SpamClassifier()

# ------------------ Key Generation and Connection ------------------

public_key, private_key = rsa.newkeys(512)

choice = input("Do you want to host (1) or connect (2): ")

if choice == "1":
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind(('localhost', 12345))
    server.listen()
    client, _ = server.accept()
    client.send(public_key.save_pkcs1())
    partner_public_key = rsa.PublicKey.load_pkcs1(client.recv(1024))

elif choice == "2":
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect(('localhost', 12345))
    partner_public_key = rsa.PublicKey.load_pkcs1(client.recv(1024))
    client.send(public_key.save_pkcs1())
else:
    exit()

# ------------------ Messaging Functions ------------------

def sending_messages(c):
    while True:
        message = input("")
        if spam_detector.is_spam(message):
            print("[!] Message blocked as spam.")
            continue
        encrypted = rsa.encrypt(message.encode(), partner_public_key)
        c.send(encrypted)

def receiving_messages(c):
    while True:
        encrypted = c.recv(1024)
        if not encrypted:
            break
        try:
            decrypted = rsa.decrypt(encrypted, private_key).decode()
            print("Partner:", decrypted)
        except:
            print("Failed to decrypt message.")

# ------------------ Threading ------------------

threading.Thread(target=sending_messages, args=(client,)).start()
threading.Thread(target=receiving_messages, args=(client,)).start()
