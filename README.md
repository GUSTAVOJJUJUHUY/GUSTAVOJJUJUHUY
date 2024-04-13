import yfinance as yf
import matplotlib.pyplot as plt

# Função para calcular a média móvel
def calcular_media_movel(data, janela):
    return data['Close'].rolling(window=janela).mean()

# Função para determinar quando comprar e vender com base no cruzamento de médias móveis
def estrategia_cruzamento_media_movel(data, curta_janela=50, longa_janela=200):
    sinais = pd.DataFrame(index=data.index)
    sinais['Preco'] = data['Close']
    sinais['MediaCurta'] = calcular_media_movel(data, curta_janela)
    sinais['MediaLonga'] = calcular_media_movel(data, longa_janela)
    sinais['SinalCompra'] = 0.0
    sinais['SinalVenda'] = 0.0
    sinais['SinalCompra'][curta_janela:] = np.where(sinais['MediaCurta'][curta_janela:] > sinais['MediaLonga'][curta_janela:], 1.0, 0.0)
    sinais['SinalVenda'][curta_janela:] = np.where(sinais['MediaCurta'][curta_janela:] < sinais['MediaLonga'][curta_janela:], -1.0, 0.0)
    return sinais

# Função principal
def main():
    # Obtendo os dados do Yahoo Finance
    ticker = 'AAPL'  # Substitua 'AAPL' pelo símbolo da ação desejada
    data = yf.download(ticker, start='2020-01-01', end='2023-01-01')

    # Aplicando a estratégia de cruzamento de médias móveis
    sinais = estrategia_cruzamento_media_movel(data)

    # Visualizando os sinais de compra e venda
    plt.figure(figsize=(10, 5))
    plt.plot(data['Close'], label='Preço de Fechamento', alpha=0.5)
    plt.plot(sinais['MediaCurta'], label='Média Móvel Curta', alpha=0.5)
    plt.plot(sinais['MediaLonga'], label='Média Móvel Longa', alpha=0.5)
    plt.plot(sinais[sinais['SinalCompra'] == 1].index, sinais['MediaCurta'][sinais['SinalCompra'] == 1], '^', markersize=10, color='g', lw=0, label='Compra')
    plt.plot(sinais[sinais['SinalVenda'] == -1].index, sinais['MediaCurta'][sinais['SinalVenda'] == -1], 'v', markersize=10, color='r', lw=0, label='Venda')
    plt.title('Estratégia de Cruzamento de Médias Móveis')
    plt.legend()
    plt.show()

if __name__ == "__main__":
    main()
