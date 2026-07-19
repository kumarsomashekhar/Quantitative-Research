The objective

Analyze the historical monthly natural gas prices.
Identify the trend and seasonality.
Extrapolate prices for one additional year.
Build a function that accepts any date and returns an estimated gas price.

Since the data is monthly and the prompt explicitly mentions considering seasonal trends, a good approach is:

Convert the dates to datetime.
Separate the trend (overall increase/decrease over time).
Capture the seasonality (monthly effects).
Use linear interpolation for dates between known monthly observations.
Forecast one additional year using the estimated trend + repeating seasonal pattern.

# ============================================
# Natural Gas Price Estimation & Forecasting
# ============================================

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from scipy.interpolate import interp1d

# -------------------------------
# Load Data
# -------------------------------
df = pd.read_csv("Nat_Gas.csv")

df["Dates"] = pd.to_datetime(df["Dates"])
df = df.sort_values("Dates")

# Create time index
df["Time"] = np.arange(len(df))

# -------------------------------
# Trend Estimation
# -------------------------------

# Fit a linear trend
coeff = np.polyfit(df["Time"], df["Prices"], 1)

trend = np.poly1d(coeff)

df["Trend"] = trend(df["Time"])

# -------------------------------
# Seasonal Component
# -------------------------------

df["Month"] = df["Dates"].dt.month

df["Seasonality"] = df["Prices"] - df["Trend"]

monthly_seasonality = (
    df.groupby("Month")["Seasonality"]
    .mean()
)

# -------------------------------
# Forecast Next 12 Months
# -------------------------------

future_dates = pd.date_range(
    start=df["Dates"].max() + pd.offsets.MonthEnd(1),
    periods=12,
    freq="M"
)

future_time = np.arange(
    len(df),
    len(df)+12
)

future_trend = trend(future_time)

future_prices = []

for i, date in enumerate(future_dates):
    seasonal = monthly_seasonality[date.month]
    future_prices.append(future_trend[i] + seasonal)

forecast = pd.DataFrame({
    "Dates": future_dates,
    "Prices": future_prices
})

# -------------------------------
# Combine Historical + Forecast
# -------------------------------

all_data = pd.concat([
    df[["Dates","Prices"]],
    forecast
]).reset_index(drop=True)

# -------------------------------
# Interpolation Function
# -------------------------------

x = all_data["Dates"].map(pd.Timestamp.toordinal)
y = all_data["Prices"]

price_function = interp1d(
    x,
    y,
    fill_value="extrapolate"
)

def estimate_price(date):

    """
    Estimate natural gas price for any date.

    Example:
        estimate_price("2023-08-15")
    """

    date = pd.Timestamp(date)

    return float(price_function(date.toordinal()))

# -------------------------------
# Visualization
# -------------------------------

plt.figure(figsize=(12,6))

plt.plot(
    df["Dates"],
    df["Prices"],
    label="Historical Prices",
    marker="o"
)

plt.plot(
    forecast["Dates"],
    forecast["Prices"],
    label="Forecast",
    linestyle="--",
    marker="o"
)

plt.title("Natural Gas Prices (Historical + Forecast)")
plt.xlabel("Date")
plt.ylabel("Price")
plt.grid(True)
plt.legend()

plt.show()

# -------------------------------
# Example
# -------------------------------

print("Estimated Price on 2023-08-15 :",
      round(estimate_price("2023-08-15"),2))

print("Estimated Price on 2025-03-20 :",
      round(estimate_price("2025-03-20"),2))


Why this approach is appropriate
Linear trend (np.polyfit) captures the overall long-term movement in gas prices.
Monthly seasonality is calculated by averaging the deviation from the trend for each month, reflecting recurring seasonal effects.
Interpolation (interp1d) estimates prices for any day between monthly observations instead of only month-end values.
Forecasting extends the trend by 12 months while reusing the learned monthly seasonal pattern, producing reasonable indicative prices.

I first explored the monthly natural gas prices to understand their behavior. Since commodity prices often exhibit both long-term trends and seasonal patterns, I decomposed the data into a linear trend and a monthly seasonal component. I estimated the trend using linear regression (numpy.polyfit) and calculated the average seasonal adjustment for each month. I then forecasted one additional year by extending the trend and applying the corresponding monthly seasonal factors. Finally, I used interpolation so that the solution could estimate the gas price for any user-provided date, not just month-end observations. This approach is simple, interpretable, and appropriate given the limited monthly dataset.

      
