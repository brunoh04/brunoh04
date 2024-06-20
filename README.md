/brunoh04
├── src
│   ├── main.py
│   ├── web_scraper.py
│   ├── data_analysis.py
│   └── storage.py
├── wsgi.py
├── requirements.txt
└── models/

requirements.txt
requests
beautifulsoup4
pandas
scikit-learn
ipfshttpclient
joblib

scraper.py
import requests
from bs4 import BeautifulSoup
import pandas as pd

def get_market_data(query):
    url = f"https://www.example.com/search?q={query}"
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')

    data = []
    for item in soup.find_all('div', class_='market-item'):
        title = item.find('h2').text
        price = item.find('span', class_='price').text
        description = item.find('p', class_='description').text.strip() if item.find('p', class_='description') else ''
        data.append({'title': title, 'price': price, 'description': description})

    df = pd.DataFrame(data)
    df.to_csv(f'data/raw/{query}_market_data.csv', index=False)
    return df
data_analysis.py
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.ensemble import RandomForestRegressor
import joblib

def analyze_market_data_linear_regression(query):
    df = pd.read_csv(f'data/raw/{query}_market_data.csv')

    df['price'] = df['price'].replace('[\$,]', '', regex=True).astype(float)

    X = df[['title']]
    y = df['price']

    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

    model = LinearRegression()
    model.fit(X_train, y_train)

    score = model.score(X_test, y_test)
    joblib.dump(model, 'models/market_model_linear_regression.pkl')

    return score

def analyze_market_data_random_forest(query):
    df = pd.read_csv(f'data/raw/{query}_market_data.csv')

    df['price'] = df['price'].replace('[\$,]', '', regex=True).astype(float)

    X = df[['title_length', 'description_length']]  # Exemplo de novas features
    y = df['price']

    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

    model = RandomForestRegressor()
    model.fit(X_train, y_train)

    score = model.score(X_test, y_test)
    joblib.dump(model, 'models/market_model_random_forest.pkl')

    return score
storage.py
import ipfshttpclient

def upload_to_ipfs(filepath):
    client = ipfshttpclient.connect('/dns/localhost/tcp/5001/http')
    res = client.add(filepath)
    return res['Hash']

def download_from_ipfs(hash):
    client = ipfshttpclient.connect('/dns/localhost/tcp/5001/http')
    client.get(hash)
main.py
from flask import Flask, jsonify, request
from scraper import get_market_data
from data_analysis import analyze_market_data_linear_regression, analyze_market_data_random_forest
from storage import upload_to_ipfs

app = Flask(__name__)

@app.route('/scrape', methods=['POST'])
def scrape_data():
    query = request.json['query']
    df = get_market_data(query)
    return jsonify({'message': 'Data scraped successfully', 'data': df.to_dict(orient='records')})

@app.route('/analyze_linear', methods=['POST'])
def analyze_data_linear():
    query = request.json['query']
    score = analyze_market_data_linear_regression(query)
    return jsonify({'message': 'Data analyzed with Linear Regression successfully', 'score': score})

@app.route('/analyze_forest', methods=['POST'])
def analyze_data_forest():
    query = request.json['query']
    score = analyze_market_data_random_forest(query)
    return jsonify({'message': 'Data analyzed with Random Forest successfully', 'score': score})

@app.route('/upload', methods=['POST'])
def upload_to_ipfs_endpoint():
    filepath = request.json['filepath']
    file_hash = upload_to_ipfs(filepath)
    return jsonify({'message': 'File uploaded to IPFS', 'hash': file_hash})

if __name__ == '__main__':
    app.run(debug=True)
wsgi.py
from main import app as application
