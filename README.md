mkdir -p market-research-ai/{data/raw,data/processed,src,models,notebooks}
cd market-research-ai
touch README.md requirements.txt src/__init__.py src/web_scraper.py src/data_analysis.py src/storage.py src/main.py
# Market Research AI

Este projeto tem como objetivo criar uma IA para realizar pesquisas de mercado de forma mais rápida, coletando dados da web, analisando-os e armazenando-os em um banco descentralizado.

## Instalação

```bash
pip install -r requirements.txt
python src/main.py

#### 2. `requirements.txt`

Cole o seguinte conteúdo:

```plaintext
requests
beautifulsoup4
pandas
scikit-learn
ipfshttpclient
joblib
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
        data.append({'title': title, 'price': price})

    df = pd.DataFrame(data)
    df.to_csv(f'data/raw/{query}_market_data.csv', index=False)
    return df
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
import joblib

def analyze_market_data(query):
    df = pd.read_csv(f'data/raw/{query}_market_data.csv')

    df['price'] = df['price'].replace('[\$,]', '', regex=True).astype(float)

    X = df[['title']]
    y = df['price']

    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

    model = LinearRegression()
    model.fit(X_train, y_train)

    score = model.score(X_test, y_test)
    joblib.dump(model, 'models/market_model.pkl')

    return score
import ipfshttpclient

def upload_to_ipfs(filepath):
    client = ipfshttpclient.connect('/dns/localhost/tcp/5001/http')
    res = client.add(filepath)
    return res['Hash']

def download_from_ipfs(hash):
    client = ipfshttpclient.connect('/dns/localhost/tcp/5001/http')
    client.get(hash)
from web_scraper import get_market_data
from data_analysis import analyze_market_data
from storage import upload_to_ipfs

def main():
    query = "example market"
    df = get_market_data(query)
    score = analyze_market_data(query)
    print(f'Model score: {score}')

    file_hash = upload_to_ipfs(f'data/raw/{query}_market_data.csv')
    print(f'Uploaded to IPFS with hash: {file_hash}')

if __name__ == "__main__":
    main()
pip install -r requirements.txt
python src/main.py
