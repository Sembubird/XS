**XScript 程式效率優化指南**

### 

## **XS 程式效率介紹**

### **1\. 程式執行效率**

* 效率是投入與產出的關係。在效果(執行結果)相同的情況下，相同投入，產出較多，則效率高；相同產出，投入較少，則效率高。

### **2\. 程式執行效率不佳的可能影響**

* 選股、警示及交易需要比較多時間。  
* 影響 XQ 系統效能，進而影響所有使用者。  
* 可能產生錯誤。

### **3\. 程式沒有效率的類型**

* 準備太多不需要的資料  
* 執行不必要的運算  
* 計算K棒數  
* 執行不需要的迴圈  
* 不斷進行多餘判斷  
* 不會使用內建函數  
* 監控不需要監控的商品

### **4\. 準備太多不需要的資料**

* 選出外資連續買超三天的股票：  
  * if TrueAll(GetField("外資買賣超","D")\>0,3) then ret=1;  
* 比較有效率的寫法：只準備1筆資料，而用預設的10筆。  
  * setTotalBar(1);  
  * if TrueAll(GetField("外資買賣超","D")\>0,3) then ret=1;

### **5\. 準備不足夠的資料**

* 必須留意資料是否為資料庫的數據。若是，準備1筆即可，若不是，需要準備足夠的資料，不然會有邏輯錯誤，價格高於過去3期均價：  
  * setTotalBar(1);  
  * if c \> average(c\[1\],3) then ret=1;  
* 以上寫法 OK，但以下寫法不行，應增加資料筆數。  
  * setTotalBar(4);  
  * if c \> average(c,3)\[1\] then ret=1;  
* 讀取資料筆數足夠時，average(c\[1\],3) 結果與 average(c,3)\[1\] 結果相同。

### **6\. 執行不必要的運算**

* 重複使用 SetTotalBar  
* 重複使用 SetBarFreq  
* 判斷均線上揚或下彎  
* 一直運算只需運算一次的工作  
* 不需要在最後一根K棒之前執行

### **7\. 重複使用 SETTOTALBAR**

* 在投資策略的讀取資料筆數時，使用不同的 SetTotalBar。  
* Delphi

setTotalBar(30);  
...  
setTotalBar(100);  
...  
setTotalBar(20);

*   
* 系統只會採用這三個 setTotalBar 中編譯成功的最大數值。

### **8\. 重複使用 SETBARFREQ**

* 在選股腳本中，欲使用下款單選單選取頻率(日、週、月)，讓使用者可以選擇查詢不同頻率的資料。  
* Delphi

input: Choice(1, "選股頻率", InputKind:=dict(\["日", 1\], \["週", 2\], \["月", 3\]));  
switch(Choice)  
begin  
    case 1: setBarFreq("D");  
    case 2: setBarFreq("W");  
    case 3: setBarFreq("M");  
end;

*   
* 系統只會採用最後一個 setBarFreq。

### **9\. 判斷均線上揚或下彎**

* 我們經常在策略當中會有判斷趨勢的指標，常用的是均線。要判斷均線是否上揚，我們會計算當根K棒的均線，然後與前一根K棒的均線比較。  
  * Condition1 \= average(c,20) \> average(c\[1\],20);  
* 其實用扣抵值判斷會比較有效率。  
  * Condition1 \= c \>= c\[20\];

### **10\. 計算K棒數 (1)**

* 網友 **Luckyguide** 希望如下圖，在每天第一根K棒繪製垂直線以區隔每天行情。  
* 網友 **GammaEco** 熱心回覆並提供以下的指標腳本。  
* Delphi

if barfreq \<\> "Min" then RaiseRunTimeError("限1分線");  
variable: KN1(0);  
variable: intrabarpersist k1(0);  
if IsSessionFirstBar \= true then k1 \= CurrentBar;  
KN1 \= (CurrentBar \- k1) \+ 1;  
if KN1 \= 1 then value1 \= 3000 else value1 \= 0;  
plot1(value1, "openH");

*   
*   
* 上述腳本可如預期繪製指標，但更有效率的寫法：  
* Delphi

if barfreq \<\> "Min" then RaiseRunTimeError("限1分線");  
if date \<\> date\[1\] then Plot1(3000, "openH") else plot1(0);

*   
* 

### **11\. 計算K棒數 (2)**

* 請撰寫1分鐘頻率的交易腳本，在連續三根K棒收紅時進場以觸發價往下減一檔的價格選進10口台指期，若連續三根K棒都沒有全部成交，則將剩餘沒成交的委託單改價為目前成交價往上加1檔。改價之後，若過三根K棒沒有成交，則繼續改價。  
* //非逐筆洗價(每分鐘洗價一次)或逐筆洗價都可以  
* Delphi

if barfreq \<\> "Min" or barinterval \<\> 1 then raiseruntimeerror("限用1分鐘");  
if GetInfo("IsRealTime") \= 0 then return;  
var: Kbar(0);

*   
*   
* **計算K棒數 (2) (較沒效率的寫法)**  
* Delphi

if Time \>= 090300 then  
begin  
    if TrueAll(c\[1\] \> o\[1\], 3) and Position \= 0 then  
    begin  
        SetPosition(10, addspread(c, \-1));  
        Kbar \= 0;  
    end;  
    if Position \<\> Filled and Position \<\> 0 then Kbar \+= 1 else Kbar \= 0;  
    if Position \<\> Filled and Kbar \= 3 then  
    begin  
        SetPosition(Position, AddSpread(c, 1));  
        Kbar \= 0;  
    end;  
end;

*   
*   
* **計算K棒數 (2) (較有效率的寫法)**  
* Delphi

if Time \>= 090300 then  
begin  
    if TrueAll(c\[1\] \> o\[1\], 3) and Position \= 0 then  
        SetPosition(10, addspread(c, \-1));  
    if Position \<\> Filled and Position \<\> 0 then  
        SetPosition(Position, addspread(c, 1));  
end;

*   
* 

### **12\. 一直運算只需運算一次的工作 (1)**

* 30分鐘頻率，判斷股票是否在前一個交易日最後半小時跌幅震盪1%以內。  
* **最沒效率的寫法：**  
* Delphi

if barfreq \<\> "Min" or barinterval \<\> 30 then raiseRunTimeError("限用30分鐘");  
var: myHigh(0), myLow(0);  
myHigh \= h\[1\];  
myLow \= l\[1\];  
condition1 \= myHigh \< myLow \* 1.01;  
if condition1 \= false then return;

*   
* 整個交易時段都在判斷，最沒效率。  
* **一樣沒效率的寫法：只在第一根K棒運算**  
* Delphi

if barfreq \<\> "Min" or barinterval \<\> 30 then raiseRunTimeError("限用30分鐘");  
var: myHigh(0), myLow(0);  
if time \= 090000 then   
begin  
    myHigh \= h\[1\];  
    myLow \= l\[1\];  
    if myHigh \< myLow \* 1.01 then condition1 \= true else return;  
end  
else return;

*   
* 雖然在第一根K棒判斷，但30分鐘的每個 Tick 都在判斷，同樣沒效率。  
* **最有效率的作法：使用 raiseRunTimeError 取代 Return，只在第一個 Tick 判斷，不符合時終止策略。**  
* Delphi

if barfreq \<\> "Min" or barinterval \<\> 30 then raiseRunTimeError("限用30分鐘");  
var: myHigh(0), myLow(0);  
myHigh \= h\[1\];  
myLow \= l\[1\];  
if myHigh \>= myLow \* 1.01 then   
    raiseRunTimeError("條件不符，中斷策略");

*   
* 

### **13\. 一直運算只需運算一次的工作 (2)**

* 網友 **阿DR5** 的腳本使用 getsymbolinfo("買賣現沖") 判斷是否可現沖。  
* 這動作會從閱覽端接到收盤一直在判斷是否可現沖。有效率的做法應該是只判斷一次，且禁止現沖的商品應停止監控。  
  * **無效率：** if getsymbolinfo("買賣現沖") \= true then begin ... end;  
  * **優化：**  
* Delphi

if getsymbolinfo("買賣現沖") \= false then raiseRunTimeError("停止監控");  
if time \= 090000 then ...  
if time \= 090100 then ...

*   
* 

### **14\. 一直運算只需運算一次的工作 (3)**

* 網友 **蘭庭王** 的腳本使用變數記錄固定不變的數值 (value1, value2, value3)。  
* 這動作造成不必要的運算，低效率。應改用參數處理。  
  * **無效率：**  
  * Delphi

value1 \= 8; //漲幅進場%  
value2 \= 6; //獲利出場%  
value3 \= 6; //停損%  
value4 \= highest(high, 20); //20天高  
value5 \= lowest(low, 60); //60天低

*   
  *   
  * **優化：**  
  * Delphi

input: change(8, "漲幅進場%"), WinPercent(6, "獲利出場%"), LossPercent(6, "停損%");  
value4 \= highest(high, 20); //20天高  
value5 \= lowest(low, 60); //60天低

*   
  * 

### **15\. 一直運算只需運算一次的工作 (4)**

* 網友 **CL** 希望在1分鐘頻率下，跨30分鐘頻率用 SMA 計算 MACD。他寫的程式修改簡化如下：  
* Delphi

if barfreq \<\> "Min" or barinterval \<\> 1 then raiseRunTimeError("限用1分鐘");  
input: FastLength(3, "DIF天數"), SlowLength(10, "DIF天數"), MACDLength(16, "MACD天數");  
var: i(0);  
array: ave1\[20\](0);  
Value1 \= Average(getField("close", "30"), FastLength) \- Average(getField("close", "30"), SlowLength); //macd line

for i \= 1 to MACDLength //共16個  
    ave1\[i\] \= xfMin\_getvalue("30", value1, i \- 1);  
Value2 \= Array\_Sum(ave1, 1, MACDLength) / MACDLength; //signal line  
Value3 \= Value1 \- Value2; //histogram  
plot1(Value3, "histogram");

*   
* 問題：每分鐘跑迴圈，計算過去16個30分鐘的 Value1 值與本期的 Value1 計算指標，只要在30分鐘還沒結束之前，過去15個 Value1 的數據並沒有改變，不需要重複一直計算。  
* **效率較好的寫法：**  
* Delphi

if barfreq \<\> "Min" or barinterval \<\> 1 then raiseRunTimeError("限用1分鐘");  
input: FastLength(3, "DIF天數"), SlowLength(10, "DIF天數"), MACDLength(16, "MACD天數");  
var: i(0), myTime(0);  
array: ave1\[20\](0);

Value1 \= Average(getField("close", "30"), FastLength) \- Average(getField("close", "30"), SlowLength); //macd line  
myTime \= getfield("時間", "30");  
if myTime \<\> myTime\[1\] then //只在30分鐘的第一分鐘，記錄過去15期的value1  
begin  
    for i \= 1 to MACDLength \- 1 //共15個  
        ave1\[i+1\] \= xfMin\_getvalue("30", value1, i);  
end;  
ave1\[1\] \= MACDLength \* value1;  
Value2 \= Array\_Sum(ave1, 1, MACDLength) / MACDLength; //signal line  
Value3 \= Value1 \- Value2; //histogram  
plot1(value3, "histogram");

*   
* 

### **16\. 不需要在最後一根K棒之前執行 (1)**

* 網友發問希望選出100天內第一次收盤創100天新高的股票。  
  * condition1 \= c cross over highest(h\[1\], 100);  
  * if condition1 and countif(condition1, 100\) \= 1 then ret=1;  
* 上述程式有兩個問題：(1) 從第一筆就在計算當下創100天新高次數，這是不需要的，只要在最後一筆計算即可。(2) 用 countif 函數計算次數是錯誤的。  
* **正確的做法是只準備一筆資料，並跑迴圈計算符合創新高的次數：**  
* Delphi

setTotalBar(1);  
var: i(0), sum(0);  
if isLastBar then  
begin  
    for i \= 0 to 100  
    begin  
        if c\[i\] \> highest(h\[i+1\], 100)  
        then begin  
            value2 \= highestBar(h\[i+1\], 100) \+ 1; //找其目前距離今幾天前大漲K棒  
            sum \+= 1;  
        end;  
        for i \= 1 to value2 \- 1 //今天不算的話，從那天開始跑  
            if c\[i\] \> value1 then sum \+= 1;  
        if sum \= 0 then ret \= 1; //今天創新高，但之前沒有  
    end;  
end;

*   
* 

### **17\. 不需要在最後一根K棒之前執行 (2)**

* 網友 **biscrewn** 在論壇詢問分鐘頻率下，用迴圈找出過去15根K棒的最高價對應當天的最高價且當天的K棒有超過30%的上影線的寫法。  
* **無效率寫法：**  
* Delphi

i \= 0;  
while i \< 15  
begin  
    if High\[i\] \= Value2 then  
    begin  
        barRange \= High\[i\] \- Low\[i\];  
        if barRange \<\> 0 then  
            upperShadow \= (High\[i\] \- MaxList(Open\[i\], Close\[i\])) / barRange;  
        break;  
    end;  
    i \+= 1;  
end;

*   
*   
* **較有效率的寫法：**  
* Delphi

var: i(0), hasRet(false);  
if islastbar then  
begin  
    value3 \= getfield("最高價", "D");  
    hasRet \= false;  
    for i \= 1 to 15  
    begin  
        if High\[i\] \= value3 and upperShadow\[i\] \> 0.3 then  
        begin  
            hasRet \= true;  
            break;  
        end;  
    end;  
    if hasRet \= true then ret=1;  
end;

*   
* 

### **18\. 執行不需要的迴圈**

* 網友 **鍵盤上的小白** 在論壇分享選股策略「今日成交量大於N日前的某K最大量」。  
* **無效率寫法：**  
* Delphi

input: ND(3, "今日成交量大於N日前的某K最大量");  
var: BigVolume(0);  
for i \= 0 to ND  
begin  
    if isLastBar \= true then  
    begin  
        for i \= 1 to ND  
        begin  
            if c\[i\] \< o\[i\] and v\[i\] \> BigVolume then  
                BigVolume \= v\[i\];  
        end;  
        if BigVolume \> 0 and v \> BigVolume then ret \= 1;  
    end;  
end;

*   
*   
* **優化寫法：不需要跑兩次迴圈。**  
* Delphi

input: ND(3, "今日成交量大於N日前的某K最大量");  
setTotalBar(ND \+ 1);  
var: BigVolume(0);  
if isLastBar \= false then  
begin  
    if c \< o and v \> BigVolume then BigVolume \= v;  
end  
else  
    if BigVolume \<\> 0 and v \> BigVolume then ret \= 1;

*   
* 

### **19\. 不斷進行多餘判斷**

* 一旦條件符合，重複執行指令，例如一直下單建立部位，或一直送出無法成交指令。  
* 有3項部位且符合出場條件時，則平倉：  
  * if condition1 and position \> 0 and filled \<\> 0 then setposition(0);  
* 10分鐘頻率，價格高於過去3期最高價碼買進1張，最多持有2張：  
* Delphi

if condition1 then   
    setposition(position \+ 1);

*   
* 在成交之前會持續增加購買量，亦即連續購買。  
* **解決方法：可設定單根K棒只觸發一次。**  
* Delphi

if barfreq \<\> "Min" or barinterval \<\> 10 then raiseRunTimeError("限用10分鐘");  
value1 \= highest(h\[1\], 3);  
condition1 \= c \> value1;  
if myTime \<\> Time and condition1 and filled \< 1 then  
begin  
    setposition(position \+ 1);  
    myTime \= Time;  
end;

*   
* 

### **20\. 不當使用內建函數 (1)**

* 使用 Highest 與 simpleHighest 基本上可得到相同結果，但 Highest 較有效率。  
* 但若資料超過500筆，則 Highest 計算結果將不正確。

### **21\. 不當使用內建函數 (2)**

* 多次呼叫相同的內建函數，是比較沒有效率的作法，應使用變數處理。  
  * **較無效率：**  
  * Delphi

if c \> getfield("收盤價", "S") and value1 \> getfield("收盤價", "S") then ret=1;

*   
  *   
  * **優化：**  
  * Delphi

value1 \= getfield("收盤價", "S");  
if c \> value1 and value2 \> value1 then ret=1;

*   
  * 

### **22\. 不當使用內建函數 (3)**

* 但在跨頻率呼叫內建函數，應避免使用變數，不然很容易有邏輯錯誤。  
* Delphi

//1分鐘頻率，價格高於30分K的收盤價  
if barfreq \<\> "Min" or barinterval \<\> 1 then raiseruntimeerror("限用1分鐘");  
if c cross over average(getfield("收盤價", "30"), 5) then ret \= 1;

*   
* 

### **23\. 監控不需要監控的商品**

* 不少人執行商品選擇全量全部商品，其實裡面很多商品根本不可能交易。這雖然與程式撰寫效率無關，但卻會影響整個程式執行的效率。  
* 比較有效率的做法是先透過選股策略篩選商品，再以特定選股監控這些商品。