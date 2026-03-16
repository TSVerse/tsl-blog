
Индикатор и код на Pine в файле C:\Users\TSLab\Desktop\GPT\gpttunnel\TSLabDoc-master\Индикаторы\Linear Regression Based Volume Profile.md
1. Нужно сделать анализ индикатора. Понять, что именно он считает и выводит на экран.
   Сделать анализ, возможно ли такой индикатор нарисовать в TSLab.
2. Задача, сделать такой же индикатор, но на C# и подгрузить dll в программу TSLab.
   Название индикатора LinReg_Claster_GPT
   Сделать скрипт и добавить индикатор. Скрипт назвать Linear Regression Volume Profile _GPT. Скрипт поместить в папку AI.
   Вывести индикатор на экран, добиться, чтобы линии были нарисованы на графике цены.
   Все параметры вывести в контрольную панель, чтобы у человека была возможность их менять на контрольной панели.
   В скрипте использовать поставщик BingXSpot инструмент BTC-USDT

Linear Regression Volume Profile plots the volume profile fixated on the linear regression of the lookback period rather than statically across y = 0. This helps identify potential support and resistance inside of the price channel.  



**Settings**  
_Linear Regression_  

- Linear Regression Source: the price source in which to sample when calculating the linear regression  
    
- Length: the number of bars to sample when calculating the linear regression  
    
- Deviation: the number of standard deviations away from the linear regression line to draw the upper and lower bounds  
    

  
_Linear Regression_  

- Rows: the number of rows to divide the linear regression channel into when calculating the volume profile  
    
- Show Point of Control: toggle whether or not to plot the level with highest amount of volume  
    

**Usage**  
Similar to the traditional Linear Regression and Volume Profile this indicator is mainly to determine levels of support and resistance. One may interpret a level with high volume (i.e. point of control) to be a potential reversal point.  
  
**Details**  
This indicator first calculates the linear regression of the specified lookback period and, subsequently, the upper and lower bound of the linear regression channel. It then divides this channel by the specified number of rows and sums the volume that occurs in each row. The volume profile is scaled to the min and max volume.  

19 апр. 2023 г.

Информация о релизе

- fixed bug regarding linear regression length being longer than bar_index  
    
- updated chart  
    

20 апр. 2023 г.

Информация о релизе

- Fixed custom Point of Control color
  
//Linear Regression Based Volume Profile © 2023 by DeltaSeek is licensed under CC BY-NC-SA 4.0 
//@version=5
indicator(title = "Linear Regression Volume Profile", overlay = true, max_bars_back = 500, max_lines_count = 500)


// CONSTANTS
// Colors 
color RED_L = #E74C3C, color RED_M = #B03A2E, color RED_D = #78281F
color ORA_L = #F0B27A, color ORA_M = #E67E22, color ORA_D = #AF601A
color YEL_L = #F7DC6F, color YEL_M = #F1C40F, color YEL_D = #B7950B
color GRE_L = #2ECC71, color GRE_M = #239B56, color GRE_D = #186A3B
color BLU_L = #85C1E9, color BLU_M = #3498DB, color BLU_D = #2874A6
color PUR_L = #8E44AD, color PUR_M = #6C3483, color PUR_D = #4A235A
color GRA_L = #BDC3C7, color GRA_M = #909497, color GRA_D = #626567
color WHI   = #FFFFFF, color BLA   = #000000, color CLE   = #00000000


// INPUTS
// Linear Regression
float   linRegSrc = input.source(close,                 "Linear Regression Source",                                 "", "",     "Linear Regression")
int     linRegLen = input.int   (100,                   "Length",                   10, 500, 1,                     "", "",     "Linear Regression")
float   linRegDev = input.float (2.0,                   "Deviation",                0.1,10,0.5,                     "", "",     "Linear Regression")

// Volume Profile
int     vpRows    = input.int   (100,                   "Rows",                     2, 100, 1,                      "", "",     "Volume Profile")
bool    showPoc   = input.bool  (true,                  "Show Point of Control",                                    "", "",     "Volume Profile")

// Appearance
color   linRegCol = input.color(color.new(GRA_M, 75),   "Linear Regression  ",                                      "", "0",    "Appearance")
color   loVolCol  = input.color(color.new(PUR_D, 25),   "Volume      Low",                                          "", "1",    "Appearance")
color   hiVolCol  = input.color(color.new(BLU_L, 25),   " High",                                                    "", "1",    "Appearance") 
color   pocCol    = input.color(RED_M,                  "Point of Control    ",                                     "", "2",    "Appearance")


// FUNCTIONS
lineSet(line l, int x1, float y1, int x2, float y2, color col, int wid = 1) =>
    line.set_xy1  (l, x1, y1)
    line.set_xy2  (l, x2, y2)
    line.set_color(l, col)
    line.set_width(l, wid)

calcVolSum(float[] sumVol, line lower, line upper) =>
    volSum = 0.0
    for j = 0 to linRegLen - 1
        if high[j] > line.get_price(upper, bar_index - j) and low[j] < line.get_price(lower, bar_index - j)
            volSum := volSum + volume[j]
    array.push(sumVol, volSum)


// CALCULATIONS
var linRegLines = array.new<line>    ()
var vpLines     = array.new<line>    ()
var vpFill      = array.new<linefill>()
var pocLine     = line.new(na, na, na, na)
sumVol          = array.new<float>   ()

linRegLen := bar_index < linRegLen ? bar_index + 1 : linRegLen

if barstate.isfirst
    for i = 1 to vpRows
        array.push(linRegLines, line.new(na, na, na, na))
        array.push(vpLines,     line.new(na, na, na, na))
        l1 = array.last(vpLines)
        array.push(vpLines,     line.new(na, na, na, na))
        l2 = array.last(vpLines)
        array.push(vpFill,      linefill.new(l1, l2, CLE))

vari  = ta.variance(linRegSrc, linRegLen)
corr  = ta.correlation(linRegSrc, bar_index, linRegLen)
stdev = ta.stdev(bar_index, linRegLen)

alph = corr * (math.sqrt(vari) / stdev)
beta = ta.sma(linRegSrc, linRegLen) - alph * ta.sma(bar_index, linRegLen)
mad  = math.sqrt(vari - vari * math.pow(corr, 2)) * linRegDev

if barstate.islast
    a = alph * (bar_index - linRegLen + 1) + beta - mad
    b = alph * bar_index + beta - mad
    
    for i = 0 to vpRows - 2
        j = i / (vpRows - 1)
        wmad = j * mad * 2
        lineSet(array.get(linRegLines, i), bar_index - linRegLen + 1, a + wmad, bar_index, b + wmad, (i == 0) or (i== vpRows - 2)  ? linRegCol : CLE)
        if i > 0        
            calcVolSum(sumVol, array.get(linRegLines, i - 1), array.get(linRegLines, i))
    
    for i = 1 to vpRows - 2
        j = (i - 1) * 2
        k = j + 1

        perc = array.get(sumVol, i - 1) / array.max(sumVol)
        lower = array.get(linRegLines, i - 1)
        upper = array.get(linRegLines, i) 

        x1 = bar_index - linRegLen + 1
        x2 = x1 + math.round(linRegLen / 4 * perc)
        lineSet(array.get(vpLines, j), x1, line.get_price(lower, x1), x2, line.get_price(lower, x2), CLE)
        lineSet(array.get(vpLines, k), x1, line.get_price(upper, x1), x2, line.get_price(upper, x2), CLE)
        linefill.set_color(array.get(vpFill, i - 1),  color.from_gradient(perc, 0.0, 1.0, loVolCol, hiVolCol))

    if showPoc
        pocIndex = array.indexof(sumVol, array.max(sumVol))
        pocLo = array.get(linRegLines, pocIndex)
        pocHi = array.get(linRegLines, pocIndex + 1)
        lineSet(pocLine, bar_index - linRegLen + 1, math.avg(line.get_y1(pocLo), line.get_y1(pocHi)), bar_index, math.avg(line.get_y2(pocLo), line.get_y2(pocHi)), pocCol, 2)