
#include "DHT.h"
#include <SPI.h>
#include <Ethernet.h>
#define DHTPIN 2
#define DHTTYPE DHT11
DHT sensor(DHTPIN, DHTTYPE);
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
byte ip[] = { 192, 168, 56, 10 };                     
byte gateway[] = { 192, 168, 56, 1 };                   
byte subnet[] = { 255, 255, 255, 0 }; 

EthernetServer server(80);
String buffer;


void setup() 
{
  pinMode(7,INPUT); // humidity
  
  pinMode(8,OUTPUT); //speaker
  pinMode(9,OUTPUT);  //speaker
  
  pinMode(3,OUTPUT); // notification led
  
  pinMode(4,OUTPUT); // power led
  pinMode(5,OUTPUT);  // power led
  
  Serial.begin(9600);
  sensor.begin();
  Ethernet.begin(mac, ip, gateway, subnet);
  server.begin();
  Serial.print("The IP Adress is : ");
  Serial.println(Ethernet.localIP( ) );
}
void loop( ) 
{ 
  int g= digitalRead(7);
  digitalWrite(4, LOW);
  digitalWrite(8, LOW);
digitalWrite(3, 170);
  
  float h = sensor.readHumidity( );
  float t = sensor.readTemperature( );
  float f = sensor.readTemperature(true);
  
  float hif = sensor.computeHeatIndex(f, h);
  float hic = sensor.computeHeatIndex(t, h, false);
  
  EthernetClient webpage = server.available();
  if (webpage) 
    {
      Serial.println("Loading RMF Webpage");
      
      boolean currentLineIsBlank = true;
      while (webpage.connected ( ) ) 
        {
          if (webpage.available ( ) ) 
            {
              char c = webpage.read ( );
              if (buffer.length() < 100) 
              {
                buffer += c;
              }
              if (c == '\n' && currentLineIsBlank) 
                {
                  Serial.println(buffer);
                  webpage.println ("HTTP/1.1 200 OK");
                  webpage.println ("Content-Type: text/html");
                  webpage.println ("Connection: close");
                  webpage.println ("Refresh: 10");
                  webpage.println ( );
                  webpage.println ("<!DOCTYPE html><html><head><style>");
                  webpage.println ("table {width: 100%;}");
                  webpage.println ("th, td { text-align: left; padding: 10px; }");
                  webpage.println ("tr:nth-child(even){ background-color: #f2f2f2 }");
                  webpage.println ("tr:nth-child(odd){ background-color: white }");
                  webpage.println ("th { background-color: #4CAF50; color: white; } </style></head>");
                  webpage.print("<body><h1><center>REMOTE MONITORING FACILITY</center></h1><h2><font color=blue>SITE 1 :  </font>");
                  webpage.print("<a href=refreshall><button> REFRESH ALL</button></h2>");
                  webpage.print("<table><tr><th>VARIABLES</th><th><center>VALUES</center></th><th><center>CONDITION</center></th></tr>"); 
                  webpage.print("<tr><td>TEMPERATURE</td><td><center>");
                  webpage.print(t);
                  webpage.print(" C / ");
                  webpage.print(f);
                  webpage.print(" F ");
                  webpage.print("</center></td><td>  </td></tr>");
                  webpage.print("<tr><td>HUMIDITY</td><td><center>");
                  webpage.print(h);
                  webpage.print(" % </center></td><td> </td></tr>");
                  webpage.print("<tr><td>HEAT INDEX</td><td><center>");
                  webpage.print(hic);
                  webpage.print(" C / ");
                  webpage.print(hif);
                  webpage.print(" F ");
                  webpage.print("</center></td><td> </td></tr>");
                  webpage.print("<tr><td>SMOKE</td><td><center>");  
                  webpage.print("N/A");
                  webpage.print(" ppm</center></td><td>  </td></tr>");
                  webpage.print("<tr><td>POWER</td><td><center>");
                  
                  if(g==1) {  
                        digitalWrite(5, 0);
                        webpage.print("<b><font color=#0bb324>PRESENT</font></b>"); 
                    }
                  else
                    { 
                      digitalWrite(5, 170);
                      webpage.print("<b><font color=red>ABSENT</font></b>");
                      //twobeep();
                    }
                    
                  webpage.println("</center></td><td>  </td></tr>");
                  
                  if (isnan(h) || isnan(t) || isnan(f)) { 
                    
                    webpage.println("<h3><font color=red>Failed to read from Temp & Humi sensor!</font></h3>");
                    threebeep();
                  }
                  else { webpage.println("<h3>Temp & Humi sensor is working properly.</h3>"); }
                  
                  webpage.println ("</table></body><br/></html>");  
                  break;

                  
                  if (buffer.indexOf("refreshall") > 0)
                  {
                        webpage.println ("Refresh");
                        onebeep();
                    }
                  buffer="";
                  
                }
                if ( c == '\n') 
                  {
                    currentLineIsBlank = true;
                  } 
                else if (c != '\r') 
                  {
                    currentLineIsBlank = false;
                  }
        }
    }
    delay(1);
    webpage.stop();
    Serial.println("Webpage disconnected");
  }
  
}
void threebeep()//abnormal alarm - 3 beep
{
  for(int p=3;p>0;p--)
  {
    digitalWrite(9,200);
    delay(100);
    digitalWrite(9,0);
    delay(100);
  }
}
void twobeep()//power alarm - 2 beep
{
  for(int p=2;p>0;p--)
  {
    digitalWrite(9,200);
    delay(100);
    digitalWrite(9,0);
    delay(100);
  }
}
void onebeep()//notification alarm - 1 beep
{

    digitalWrite(9,200);
    delay(100);
    digitalWrite(9,0);
    delay(100);

}
