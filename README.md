# Detector de direção do som

<img src="imagens\rosto.jpeg">

### Descrição

Este é um projeto de arduino que simula a audição q vem de direções diferentes de uma criatura q tem uma carinha.

| Componentes |
| :--------- |
| Arduino Uno |
| Buzzer passivo 5V |
| Display lcd 16x2 |
| I2C |
| Motor de passo 28byj-48 com drive |
| 2x sensor de som |
| resistor 330 Ohm |

## Código

```cpp

#include <Stepper.h>
#include <LiquidCrystal_I2C.h>
#include <Wire.h>

// Variáveis para ações
#define DELTA_D 23000
#define TEMPO_E 3000
#define DELTA_M 3000
#define DELTA_P 1500
#define TEMPO_M 100
#define TEMPO_P 100

LiquidCrystal_I2C lcd(0x27, 16, 2);

byte boca_centro_esq[8] = {B00000, B00000, B00000, B00000, B00000, B00001, B00010, B11100};
byte boca_centro_dir[8] = {B00000, B00000, B00000, B00000, B00000, B10000, B01000, B00111};
byte boca_dir_baixo[8] = {B00010, B00011, B00001, B00001, B00011, B00010, B00100, B11000};
byte boca_esq_baixo[8] = {B01000, B11000, B10000, B10000, B11000, B01000, B00100, B00011};
byte boca_dir_cima[8] = {B00000, B00000, B00000, B00000, B00000, B00000, B00000, B00010};
byte boca_esq_cima[8] = {B00000, B00000, B00000, B00000, B00000, B00000, B00000, B01000};

byte olho[8] = {B00000, B00000, B00000, B01110, B10011, B10001, B10001, B01110};
byte olho_piscando[8] = {B00000, B00000, B00000, B00000, B00000, B00000, B00000, B11111};

bool piscando = false, dormindo = false;
long ti_p = 0, ti_d = 0, ti_m = 0;
const int numero_passos_giro = 200, passos_por_ciclo = 4, max_pass = 50, giro_maximo = 100;
int movendo = -1;
int n_passos = 0, pos_atual = 0;

Stepper motor(numero_passos_giro, 8, 10, 9, 11);

void setup ()
{
  Serial.begin(9800);
  lcd.init();
  lcd.backlight();

  lcd.createChar(1, boca_esq_cima);
  lcd.createChar(0, boca_dir_cima);
  lcd.createChar(2, boca_esq_baixo);
  lcd.createChar(3, boca_centro_esq);
  lcd.createChar(4, boca_centro_dir);
  lcd.createChar(5, boca_dir_baixo);
  
  // Olhos
  lcd.createChar(6, olho);
  lcd.createChar(7, olho_piscando);

  lcd.setCursor(6, 0);
  lcd.write(0);
  lcd.setCursor(9, 0);
  lcd.write(1);
  lcd.setCursor(6, 1);
  lcd.write(2);
  lcd.setCursor(7, 1);
  lcd.write(3);
  lcd.setCursor(8, 1);
  lcd.write(4);
  lcd.setCursor(9, 1);
  lcd.write(5);
  lcd.setCursor(10, 0);
  lcd.write(6);
  lcd.setCursor(5, 0);
  lcd.write(6);

  pinMode(6, INPUT);
  pinMode(5, INPUT);

  pinMode(7, OUTPUT);

  randomSeed(analogRead(A2));

  motor.setSpeed(120);
}

void loop ()
{
  long t = millis();

  /* Os microfones mandam HIGH quando nenhum som é detectado
  e LOW quando, caso contrário. */
  int direita = 1 - digitalRead(6);
  int esquerda = 1 - digitalRead(5);

 if (direita == HIGH || esquerda == HIGH)
  {
    delay(100);
    ti_d = t;
    n_passos = 0;

    // Acorda o robô
    if (dormindo == true)
    {
      dormindo = false;
      lcd.backlight();
      lcd.setCursor(5, 0);
      lcd.write(6);
      lcd.setCursor(10, 0);
      lcd.write(6);
      delay(300);
      lcd.setCursor(5, 0);
      lcd.write(7);
      lcd.setCursor(10, 0);
      lcd.write(7);
      delay(100);
      lcd.setCursor(5, 0);
      lcd.write(6);
      lcd.setCursor(10, 0);
      lcd.write(6);
      delay(100);
      lcd.setCursor(5, 0);
      lcd.write(7);
      lcd.setCursor(10, 0);
      lcd.write(7);
      delay(100);
      lcd.setCursor(5, 0);
      lcd.write(6);
      lcd.setCursor(10, 0);
      lcd.write(6);
      delay(200);
      tone(7, 1500, TEMPO_M);
      delay(TEMPO_M);
      tone(7, 900, TEMPO_M);
    }
    else
    {
      if (esquerda == HIGH && direita == LOW)
      {
        movendo = 0;
      }
      else if (direita == HIGH && esquerda == LOW)
      {
        movendo = 1;
      }
      else
      {
        movendo = -1;
      }
    }
  }

  if (movendo == 0)
  {
    if (pos_atual <= giro_maximo)
    {
      Serial.println("esquerda");
      n_passos++;
      pos_atual++;
      if (n_passos > max_pass)
      {
        n_passos = 0;
        movendo = -1;
      }
      else
      {
        motor.step(passos_por_ciclo);
      }
    }
    else
    {
      movendo = -1;
    }
  }
  else if (movendo == 1)
  {
    if (pos_atual >= -giro_maximo)
    {
      pos_atual--;
      Serial.println("direita");
      n_passos--;
      if (n_passos < -max_pass)
      {
        n_passos = 0;
        movendo = -1;
      }
      else
      {
        motor.step(-passos_por_ciclo);
      }
    }
    else
    {
      movendo = -1;
    }
  }

  // Ações que não dependem de input
  if (t - ti_d >= DELTA_D && dormindo == false)
  {
    dormindo = true;
    ti_d = t;
    lcd.noBacklight();
    lcd.setCursor(5, 0);
    lcd.write(7);
    lcd.setCursor(10, 0);
    lcd.write(7);
  }
  if (t - ti_p >= DELTA_P && dormindo == false && piscando == false)
  {
    ti_p = t;
    piscando = true;
    lcd.setCursor(5, 0);
    lcd.write(7);
    lcd.setCursor(10, 0);
    lcd.write(7);
  }
  if (piscando == true && dormindo == false && t- ti_p >= TEMPO_P)
  {
    piscando = false;
    ti_p = t;
    lcd.setCursor(5, 0);
    lcd.write(6);
    lcd.setCursor(10, 0);
    lcd.write(6);
  }

  // Parte do código que faz o som
  if (dormindo == false && t - ti_m >= DELTA_M)
  {
    // se rand estiver entre 300 e 185, o buzzer irá fazer um som
    ti_m = t;
    long rand = random(300);
    if (rand >= 185)
    {
      tone(7, 1500, TEMPO_M);
      delay(TEMPO_M);
      tone(7, 900, TEMPO_M);
    }
  }
}

```

## Vídeo do projeto funcionando

## Circuito

<img src="imagens\Circuito.jpg">
Obs: o wokwi (e tinkercad) não tem microfone, então usei potenciômetros pra representar 
