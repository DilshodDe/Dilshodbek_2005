//+------------------------------------------------------------------+
//|                                                    GoldAgent.mq5 |
//|                        MetaTrader 5 Expert Advisor Template      |
//+------------------------------------------------------------------+
#property strict

// --- Input parametrlar
input double LotSize = 0.01;       // Lot hajmi
input double TakeProfit = 100;     // Take Profit (punktlarda)
input double StopLoss = 50;        // Stop Loss (punktlarda)
input int CloseHour = 23;          // Savdolarni yopish vaqti (soat)

// --- Global o'zgaruvchilar
datetime LastTradeTime = 0;

//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
int OnInit()
  {
   Print("Gold Agent Expert Advisor ishlashga tayyor.");
   return(INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Expert tick function                                             |
//+------------------------------------------------------------------+
void OnTick()
  {
   // Sessiyalarni filtr qilish
   if(!IsTradingSession()) return;

   // Narx harakati signallarini tekshirish
   if(TimeCurrent() - LastTradeTime > 60) // Savdo cheklovi
     {
      double Ask = SymbolInfoDouble(_Symbol, SYMBOL_ASK);
      double Bid = SymbolInfoDouble(_Symbol, SYMBOL_BID);

      // BUY signal (oddiy ko'rsatkichsiz narx tekshiruvi)
      if(Bid > iLow(_Symbol, PERIOD_M15, 1))
        {
         OpenTrade(ORDER_TYPE_BUY, LotSize, Ask, StopLoss, TakeProfit);
        }

      // SELL signal
      if(Ask < iHigh(_Symbol, PERIOD_M15, 1))
        {
         OpenTrade(ORDER_TYPE_SELL, LotSize, Bid, StopLoss, TakeProfit);
        }
     }
   // Kun oxirida yopish
   CloseTradesAtDayEnd();
  }
//+------------------------------------------------------------------+
//| Yordamchi funksiyalar                                           |
//+------------------------------------------------------------------+
void OpenTrade(ENUM_ORDER_TYPE type, double lot, double price, double sl, double tp)
  {
   trade.PositionOpen(_Symbol, type, lot, price, sl, tp, "GoldAgent EA");
   LastTradeTime = TimeCurrent();
  }

void CloseTradesAtDayEnd()
  {
   datetime now = TimeCurrent();
   if(Hour() >= CloseHour)
     {
      trade.PositionCloseAll();
      Print("Barcha pozitsiyalar kun oxirida yopildi.");
     }
  }

bool IsTradingSession()
  {
   int hour = Hour();
   if((hour >= 7 && hour <= 11) || (hour >= 13 && hour <= 17)) // London va Nyu-York
      return true;
   return false;
  }
