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
