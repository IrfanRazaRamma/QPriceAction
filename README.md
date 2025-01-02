# QPriceAction
QPriceAction
Optimal Buy Price Prediction Using D-Wave Quantum Computing
Overview
This project utilizes D-Wave’s quantum computing platform to optimize buy price predictions for a set of stock prices. The main goal is to identify the most optimal buy prices by applying a Binary Quadratic Model (BQM) to simulate decision-making with the stock's historical price data. The model simulates a simple trading strategy, where "Buy" signals are generated, and D-Wave’s quantum sampler helps find the best prices to buy.

Key Features:
Data Simulation: We generate buy signals at arbitrary intervals (every odd index).
Quantum Optimization: Using D-Wave’s quantum computing resources, we solve a Binary Quadratic Model (BQM) to maximize our profit and minimize penalties for selling.
Optimal Buy Price: The program calculates and outputs the most optimal price to buy based on the BQM’s solution.
Getting Started
Prerequisites
Before you run the code, ensure that you have the following installed:

Python 3.x: The code is written in Python and requires Python version 3.x or above.
Required Libraries:
pandas: Used for data manipulation and analysis.
dimod: Provides functionality for creating and solving the Binary Quadratic Model (BQM).
dwave-system: Contains the sampler for interacting with the D-Wave quantum computer.
You can install the required libraries using pip:


pip install pandas dimod dwave-system
Setup
D-Wave Access:

You will need access to a D-Wave quantum computing system. If you don’t have one, you can sign up for a free account at D-Wave and obtain an API key.
The DWaveSampler class in the dwave-system library interacts with the D-Wave system. You will need to set up authentication for the D-Wave API in your environment, typically by setting your API token as an environment variable.
Dataset:

The dataset used in this example consists of stock prices over a period of time. It is simulated data, where the "Buy" signals are artificially set at odd indices, and there are no "Sell" signals for simplicity.
How It Works
1. Data Setup
The first step is to prepare the data. In this example, we simulate a dataset containing stock prices with timestamps (Date) and closing prices (Close). We assume "Buy" signals are triggered for every odd index in the data. No "Sell" signals are considered for simplicity.


data = {
    "Date": ["2024.03.05 16:45", "2024.03.05 17:00", ...],
    "Close": [8.94, 8.96, 8.55, 8.46, 8.49, 8.59, ...],
}
2. Creating the BQM (Binary Quadratic Model)
The core of the optimization is the creation of a Binary Quadratic Model (BQM). This model is built using the dimod library, where:

"Buy" signals are assigned a negative weight (reward for buying).
"Sell" signals are assigned a positive weight (penalty for selling).
Interaction terms between "Buy" and "Sell" are added to avoid conflicting actions.


bqm = dimod.BinaryQuadraticModel.empty(vartype='BINARY')

# Adding Buy/Sell signals to the BQM
for i in range(1, len(df)):
    if df['Buy_Signal'].iloc[i]:
        bqm.add_variable(f'buy_{i}', -df['Close'].iloc[i])
    if df['Sell_Signal'].iloc[i]:
        bqm.add_variable(f'sell_{i}', df['Close'].iloc[i])

# Adding interaction between Buy and Sell signals
for i in range(1, len(df)):
    if df['Buy_Signal'].iloc[i] and df['Sell_Signal'].iloc[i]:
        bqm.add_interaction(f'buy_{i}', f'sell_{i}', 1.0)
3. Solving the BQM with Quantum Sampling
Once the BQM is set up, we use D-Wave’s EmbeddingComposite and DWaveSampler to solve it. This leverages quantum optimization to find the best combination of "Buy" signals that maximize the reward (minimize the cost) for purchasing stocks.


sampler = EmbeddingComposite(DWaveSampler())
response = sampler.sample(bqm, num_reads=100)
4. Extracting Optimal Buy Prices
After obtaining the quantum solution, we extract the optimal buy prices by iterating through the samples and identifying which buy signals were active (value = 1).


def get_optimal_buy_prices(response, df):
    optimal_buy_prices = []
    for sample, energy in response.data(['sample', 'energy']):
        for key, value in sample.items():
            if 'buy_' in key and value == 1:
                index = int(key.split('_')[1])
                optimal_buy_prices.append(df['Close'].iloc[index])
    return optimal_buy_prices
    
5. Result
The program calculates and outputs the most optimal buy price based on the quantum optimization results, as well as the lowest price for entry.


optimal_buy_prices = get_optimal_buy_prices(response, df)
best_entry_price = min(optimal_buy_prices)
print(f"Optimal Buy Prices: {optimal_buy_prices}")
print(f"The optimal entry price to buy is: {best_entry_price}")
Output
When you run the script, it will print the optimal buy prices and the best entry price based on the quantum optimization process.


Optimal Buy Prices: [8.96, 8.46, 8.53, 8.71, 8.47]
The optimal entry price to buy is: 8.46
This output indicates the best prices at which to make your buy decision, maximizing the opportunity for profit.

Conclusion
This project demonstrates how quantum computing can be applied to stock trading for decision optimization. By leveraging D-Wave's quantum sampler, we created a Binary Quadratic Model to simulate and optimize buy signals for stock prices. This is just a basic implementation, but it can be expanded to include more sophisticated trading strategies, sell signals, and other factors.

Future Work
Incorporating Sell Signals: This model currently ignores sell signals. Future work could include adding sell signal logic to create a full buy/sell strategy.
Real-World Data: The current data is simulated. Real-world stock market data could be incorporated to apply this strategy to live trading.
More Complex Models: Additional financial indicators could be included in the BQM to improve decision-making.

License
This project is licensed under the MIT License - see the LICENSE file for details.

Acknowledgments
Special thanks to D-Wave for providing access to quantum computing resources through their API.
