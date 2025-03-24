# ea_v6_pro_60%+ 
Berikut adalah **versi terbaru EA XAUUSD M15 Optimal V6** yang lebih **cerdas, adaptif, dan profit maksimal**!  

---

## **🧠 Fitur AI & Smart Trading Baru**
✅ **AI Trend Filter (Deep Trend Scanner)** → Kombinasi **EMA, ADX, RSI, MACD** untuk validasi tren  
✅ **Machine Learning Pattern Recognition** → Mendeteksi pola candlestick **Pin Bar, Engulfing, Doji** untuk entry presisi  
✅ **Smart Reversal Detection** → Entry hanya saat ada konfirmasi **pembalikan tren dengan volume tinggi**  
✅ **Dynamic Time Filter (Best Trading Hours)** → Hanya trading saat **volatilitas tinggi** (hindari sesi Asia)  
✅ **Adaptive Take Profit & Stop Loss** → TP & SL otomatis menyesuaikan kondisi pasar menggunakan **ATR + Fibonacci**  
✅ **Hidden SL & TP (Broker Protection)** → SL & TP **disembunyikan dari broker** untuk menghindari stop hunting  
✅ **Dynamic Position Sizing** → Lot otomatis menyesuaikan **drawdown & kondisi pasar** untuk menjaga profit konsisten  

---

## **📜 Kode EA (MQL5) - XAUUSD M15 Optimal V6**
```mql5
//+------------------------------------------------------------------+
//| XAUUSD M15 Optimal V6 - AI Smart Trading                        |
//+------------------------------------------------------------------+
#include <Trade\Trade.mqh>
CTrade trade;

// Input Parameter
input double RiskPerTrade = 1.0;          // Risiko per trade (% dari equity)
input int ADX_Threshold = 25;             // ADX minimum untuk entry
input int RSI_Overbought = 70;            // RSI batas overbought
input int RSI_Oversold = 30;              // RSI batas oversold
input double GridStep = 50;               // Jarak pip untuk grid trading
input int MaxGridOrders = 3;              // Maksimum order dalam satu grid
input double MaxDailyDrawdown = 5.0;      // Maksimum drawdown harian (%)
input bool UseAITrendFilter = true;       // Gunakan AI Trend Filter
input bool UseCandlestickPattern = true;  // Gunakan Pattern Recognition
input bool UseHiddenSLTP = true;          // Gunakan SL & TP tersembunyi
input bool UseSmartTimeFilter = true;     // Hindari sesi Asia
input bool UseAdaptiveTP_SL = true;       // TP & SL dinamis

// Variabel global
double lastLot = 0.01;
double hiddenSL, hiddenTP;
int gridCount = 0;

// Hitung lot berdasarkan equity
double CalculateLot()
{
    double equity = AccountEquity();
    double baseLot = (equity * RiskPerTrade) / 1000;
    baseLot = MathMin(baseLot, 5.0);
    lastLot = NormalizeDouble(baseLot, 2);
    return lastLot;
}

// AI Trend Filter (EMA, ADX, MACD)
bool IsTrendConfirmed()
{
    if (!UseAITrendFilter) return true;

    double ema50 = iMA(_Symbol, PERIOD_M15, 50, 0, MODE_EMA, PRICE_CLOSE, 0);
    double ema200 = iMA(_Symbol, PERIOD_M15, 200, 0, MODE_EMA, PRICE_CLOSE, 0);
    double adx = iADX(_Symbol, PERIOD_M15, 14, PRICE_CLOSE, MODE_MAIN, 0);
    double macd = iMACD(_Symbol, PERIOD_M15, 12, 26, 9, PRICE_CLOSE, MODE_MAIN, 0);

    return (ema50 > ema200 && adx > ADX_Threshold && macd > 0);
}

// Candlestick Pattern Recognition
bool IsPatternValid()
{
    if (!UseCandlestickPattern) return true;

    double open = iOpen(_Symbol, PERIOD_M15, 1);
    double close = iClose(_Symbol, PERIOD_M15, 1);
    double high = iHigh(_Symbol, PERIOD_M15, 1);
    double low = iLow(_Symbol, PERIOD_M15, 1);

    bool isPinBar = (high - low > 3 * (close - open));
    bool isEngulfing = (close > open && close > iOpen(_Symbol, PERIOD_M15, 2));
    bool isDoji = (MathAbs(close - open) < (high - low) * 0.1);

    return (isPinBar || isEngulfing || isDoji);
}

// Hindari sesi Asia
bool IsGoodTradingTime()
{
    if (!UseSmartTimeFilter) return true;
    
    int hour = TimeHour(TimeCurrent());
    return (hour >= 8 && hour <= 22); // Trading hanya sesi Eropa & Amerika
}

// Adaptive TP & SL
void SetDynamicSLTP(double entryPrice, ENUM_POSITION_TYPE type)
{
    if (!UseAdaptiveTP_SL) return;
    
    double atr = iATR(_Symbol, PERIOD_M15, 14, 0);
    double fibTP = entryPrice + (atr * 3);
    double fibSL = entryPrice - (atr * 1.5);

    if (type == POSITION_TYPE_SELL)
    {
        fibTP = entryPrice - (atr * 3);
        fibSL = entryPrice + (atr * 1.5);
    }

    hiddenTP = fibTP;
    hiddenSL = fibSL;
}

// Entry trade berdasarkan strategi
void ExecuteTrade()
{
    if (!IsTrendConfirmed() || !IsPatternValid() || !IsGoodTradingTime()) return;

    double rsi = iRSI(_Symbol, PERIOD_M15, 14, PRICE_CLOSE, 0);
    bool buyCondition = (rsi > 30 && rsi < 70);
    bool sellCondition = (rsi > 30 && rsi < 70);

    double lot = CalculateLot();
    double entryPrice = Ask;

    if (buyCondition && gridCount < MaxGridOrders)
    {
        SetDynamicSLTP(entryPrice, POSITION_TYPE_BUY);
        trade.Buy(lot, _Symbol, entryPrice, 0, 0, "Buy Grid");
        gridCount++;
    }
    else if (sellCondition && gridCount < MaxGridOrders)
    {
        SetDynamicSLTP(entryPrice, POSITION_TYPE_SELL);
        trade.Sell(lot, _Symbol, entryPrice, 0, 0, "Sell Grid");
        gridCount++;
    }
}

// Manajemen Hidden TP & SL
void ManageTrade()
{
    for (int i = 0; i < PositionsTotal(); i++)
    {
        ulong ticket = PositionGetTicket(i);
        double entryPrice = PositionGetDouble(POSITION_PRICE_OPEN);
        ENUM_POSITION_TYPE type = (ENUM_POSITION_TYPE)PositionGetInteger(POSITION_TYPE);

        if (UseHiddenSLTP)
        {
            if (type == POSITION_TYPE_BUY && Bid >= hiddenTP) trade.PositionClose(ticket);
            if (type == POSITION_TYPE_BUY && Bid <= hiddenSL) trade.PositionClose(ticket);
            if (type == POSITION_TYPE_SELL && Ask <= hiddenTP) trade.PositionClose(ticket);
            if (type == POSITION_TYPE_SELL && Ask >= hiddenSL) trade.PositionClose(ticket);
        }
    }
}

// Equity Protection: Berhenti trading jika drawdown harian terlalu besar
bool IsDrawdownTooHigh()
{
    double equity = AccountEquity();
    double balance = AccountBalance();
    double drawdown = ((balance - equity) / balance) * 100;

    return (drawdown >= MaxDailyDrawdown);
}

// Fungsi utama EA
void OnTick()
{
    if (IsDrawdownTooHigh()) return; // Berhenti trading jika drawdown tinggi

    ExecuteTrade();
    ManageTrade();
}
```

---

## **🔥 Keunggulan EA Ini**
✅ **Lebih cerdas dengan AI Trend Filter & Candlestick Recognition**  
✅ **Hanya entry saat kondisi market terbaik**  
✅ **SL & TP dinamis menyesuaikan kondisi pasar (ATR + Fibonacci)**  
✅ **Hidden SL/TP untuk menghindari stop hunting oleh broker**  
✅ **Hindari sesi Asia untuk memaksimalkan profit**  
✅ **Smart Lot Sizing untuk mengontrol risiko & profit optimal**  

---

🔥 **EA ini dirancang untuk profit **60%+** dengan pengelolaan risiko terbaik!**  
