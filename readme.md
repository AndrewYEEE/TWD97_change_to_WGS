#TWD97_change_to_WGS
由於之前製作某專案要用到，所以就研究一下這東西，順便寫成Android Code

##說明:
  
  簡單來說就是目前台灣的開放資料所提供的位置大部分使用GIS座標(在台灣叫:TWD97(Taiwan Datum 1997))，而非一般的WGS84(經緯度)，因此若要在map定位，必須將TWD97轉換成WGS84。
  [參考網站](http://blog.ez2learn.com/2009/08/15/lat-lon-to-twd97/)

##以下是重要code部分:
  
  
        public class TMParameter{
                  /*
                                 lat是我們輸入的緯度，
                                 long是我們輸入的經度，
                                 接著就是關於投影的一些參數，
                                 a是地球赤道半徑(Equatorial Radius)，
                                 而b是兩極半徑(Polar Radius)，
                                 而long0是指中央經線，也就是我所說的卷紙接觸地球的線，
                                 而k0是延著long0的縮放比例，長度單位都是公尺，
                                 而角度都是用弧度，我之前一直算錯就有因為是用成角度來計算，全部都要是弧度才對，我們在此用的參數是
                                 a = 6378137.0公尺
                                 b = 6356752.314245公尺
                                 long0 = 121度
                                 k0 = 0.9999
                          */
                          /*
                                  文中有提到ko這個參數，恐怕很多人搞不清楚是甚麼東西。
                                  ko的全名是中央經線尺度比。
                                  在TM投影中常用的帶寬是2度，3度，6度。
                                  因為不同的帶寬就會產生不同的變形量(因為把圓弧伸展到平面上必然的會產生變形)。
                                  中央經線就橫切面來看，就是那一條圓弧和平面接觸的點。
                                  沿著中央經線，尺度比是1.0，越向兩側變形量就越大。為了讓整幅圖可以達到一個比較平均的變形量，就加上一個尺度比。讓圓弧和投影面從切線變成割線。
                                  尺度比一般規定
                                  2度 0.9999
                                  3度 1.0000
                                  6度 0.9996
                                  尺度比通常出現在美系地圖。
                                  中國大陸叫高斯克呂格投影，是不用尺度比的。
                           */
                  public double getDx(){
                      return 250000;
                  }
                  public double getDy(){
                      return 0;
                  }
                  public double getLon0(){
                      return 121 * Math.PI / 180;
                  }
                  public double getK0(){
                      return 0.9999;
                  }
                  public double getA(){
                      return 6378137.0;
                  }
                  public double getB(){
                      return 6356752.314245;
                  }
              }

    public double[] TWD97_convert_to_WGS84(TMParameter tm, double x, double y) {
            double dx = tm.getDx();
            double dy = tm.getDy();
            double lon0 = tm.getLon0();
            double k0 = tm.getK0();
            double a = tm.getA();
            double b = tm.getB();
            //double e = tm.getE();
            double e= Math.pow((1- Math.pow(b,2)/ Math.pow(a,2)), 0.5);
            x -= dx;
            y -= dy;

            // Calculate the Meridional Arc
            double M = y/k0;

            // Calculate Footprint Latitude
            double mu = M/(a*(1.0 - Math.pow(e, 2)/4.0 - 3* Math.pow(e, 4)/64.0 - 5* Math.pow(e, 6)/256.0));
            double e1 = (1.0 - Math.pow((1.0 - Math.pow(e, 2)), 0.5)) / (1.0 + Math.pow((1.0 - Math.pow(e, 2)), 0.5));

            double J1 = (3*e1/2 - 27* Math.pow(e1, 3)/32.0);
            double J2 = (21* Math.pow(e1, 2)/16 - 55* Math.pow(e1, 4)/32.0);
            double J3 = (151* Math.pow(e1, 3)/96.0);
            double J4 = (1097* Math.pow(e1, 4)/512.0);

            double fp = mu + J1* Math.sin(2*mu) + J2* Math.sin(4*mu) + J3* Math.sin(6*mu) + J4* Math.sin(8*mu);

            // Calculate Latitude and Longitude

            double e2 = Math.pow((e*a/b), 2);
            double C1 = Math.pow(e2* Math.cos(fp), 2);
            double T1 = Math.pow(Math.tan(fp), 2);
            double R1 = a*(1- Math.pow(e, 2))/ Math.pow((1- Math.pow(e, 2)* Math.pow(Math.sin(fp), 2)), (3.0/2.0));
            double N1 = a/ Math.pow((1- Math.pow(e, 2)* Math.pow(Math.sin(fp), 2)), 0.5);

            double D = x/(N1*k0);

            // lat
            double Q1 = N1* Math.tan(fp)/R1;
            double Q2 = (Math.pow(D, 2)/2.0);
            double Q3 = (5 + 3*T1 + 10*C1 - 4* Math.pow(C1, 2) - 9*e2)* Math.pow(D, 4)/24.0;
            double Q4 = (61 + 90*T1 + 298*C1 + 45* Math.pow(T1, 2) - 3* Math.pow(C1, 2) - 252*e2)* Math.pow(D, 6)/720.0;
            double lat = fp - Q1*(Q2 - Q3 + Q4);

            // long
            double Q5 = D;
            double Q6 = (1 + 2*T1 + C1)* Math.pow(D, 3)/6;
            double Q7 = (5 - 2*C1 + 28*T1 - 3* Math.pow(C1, 2) + 8*e2 + 24* Math.pow(T1, 2))* Math.pow(D, 5)/120.0;
            double lon = lon0 + (Q5 - Q6 + Q7)/ Math.cos(fp);

            return new double[] {Math.toDegrees(lat), Math.toDegrees(lon)};
        }
