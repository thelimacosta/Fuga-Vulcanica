#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2); 

int jogadorAtual = 0;

const int pinbotao[4] = {2, 3, 4, 5};

const int pinoLeds[30] = {
  22, 23, 24, 25, 26, 27, 28, 29, 30, 31,
  32, 33, 34, 35, 36, 37, 38, 39, 40, 41,
  42, 43, 44, 45, 46, 47, 48, 49, 50, 51
};
int posicoes[4] = {0, 0, 0, 0};
const int MAX_CASAS = 30;

void resetarJogo();
void mostrarVezNoLCD();
int girarDado(int jogador);
void esperarsoltar(int jogador);
void efeitoDadoGirando();
void mostrarStatusNoLCD(String l1, String l2);
bool processarMovimento(int jogador, int dado);
void atualizarTabuleiroFisico();
void processarVitoria(int vencedor);

void setup() {
  Serial.begin(9600);
  
  for (int i = 0; i < 4; i++) {
    pinMode(pinbotao[i], INPUT_PULLUP);
  }

  for (int i = 0; i < 30; i++) {
    pinMode(pinoLeds[i], OUTPUT);
    digitalWrite(pinoLeds[i], LOW); 
  }

  lcd.init();
  lcd.backlight();

  randomSeed(analogRead(A0));

  resetarJogo(); 
}

void loop() {
  mostrarVezNoLCD(); 

  int valorDoDado = girarDado(jogadorAtual); 

  if (valorDoDado > 0) { 
    bool jogoContinua = processarMovimento(jogadorAtual, valorDoDado);
    
    if (jogoContinua) {
      jogadorAtual = (jogadorAtual + 1) % 4;
    }
  }
}

void resetarJogo() {
  for (int i = 0; i < 4; i++) posicoes[i] = 0;
  jogadorAtual = 0;
  atualizarTabuleiroFisico();
  
  lcd.clear();
  lcd.print("NOVO JOGO!");
  lcd.setCursor(0, 1);
  lcd.print("BOM JOGO A TODOS");
  delay(2000);
}

void mostrarVezNoLCD() {
  lcd.setCursor(0, 0);
  lcd.print("VEZ DO JOGADOR ");
  lcd.print(jogadorAtual + 1);
  lcd.setCursor(0, 1);
  lcd.print("Casa Atual: ");
  lcd.print(posicoes[jogadorAtual]);
}

int girarDado(int jogador) {
  if (digitalRead(pinbotao[jogador]) == LOW) {
    efeitoDadoGirando();
    int resultado = random(1, 7);
    esperarsoltar(jogador);
    return resultado;
  }
  return 0; 
}

void esperarsoltar(int jogador) {
  delay(100);
  while (digitalRead(pinbotao[jogador]) == LOW) {
    delay(10);
  }
  delay(100); 
}

void efeitoDadoGirando() {
  for (int i = 0; i < 6; i++) {
    int numFalso = random(1, 7);
    char txt[17];
    sprintf(txt, "Girando... [%d]", numFalso);
    mostrarStatusNoLCD("Lancando Dado", txt);
    delay(100);
  }
}

void mostrarStatusNoLCD(String l1, String l2) {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(l1);
  lcd.setCursor(0, 1);
  lcd.print(l2);
}

bool processarMovimento(int jogador, int dado) {
  char linha1[17];
  char linha2[17];
  
  sprintf(linha1, "Dado: %d", dado);
  sprintf(linha2, "Ande %d casas", dado);
  mostrarStatusNoLCD(linha1, linha2);
  delay(1500); 

  posicoes[jogador] += dado;

  if (posicoes[jogador] >= MAX_CASAS) {
    posicoes[jogador] = MAX_CASAS; 
    atualizarTabuleiroFisico();
    processarVitoria(jogador);
    return false;
  } else {
    atualizarTabuleiroFisico();
    sprintf(linha1, "Jogador %d foi para", jogador + 1);
    sprintf(linha2, "Casa: %d", posicoes[jogador]);
    mostrarStatusNoLCD(linha1, linha2);
    delay(1500);
    return true;
  }
}

void processarVitoria(int vencedor) {
  char msgVencedor[17];
  sprintf(msgVencedor, " JOGADOR %d VENCEU!", vencedor + 1);
  mostrarStatusNoLCD("  PARABENS!!!", msgVencedor);
  
  for (int v = 0; v < 5; v++) {
    lcd.noBacklight();
    for (int i = 0; i < 30; i++) digitalWrite(pinoLeds[i], LOW);
    delay(250);
    
    lcd.backlight();
    for (int i = 0; i < 30; i++) digitalWrite(pinoLeds[i], HIGH);
    delay(250);
  }

  delay(2000);
  resetarJogo();
}

void atualizarTabuleiroFisico() {
  for (int i = 0; i < 30; i++) {
    digitalWrite(pinoLeds[i], LOW);
  }

  for (int p = 0; p < 4; p++) {
    if (posicoes[p] > 0) {
      int indiceLed = posicoes[p] - 1;
      digitalWrite(pinoLeds[indiceLed], HIGH);
    }
  }
}
