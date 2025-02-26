#define arrLen 8 //This defines the length of the long array which is randomly generated and inputted into the CRC
long longArr[arrLen];
long longArrFlip[arrLen];
int clk = 2;
int dat = 3;
int ot0 = 4;
int ot1 = 5;
int ot2 = 6;
int ot3 = 7;
int ot4 = 8;
int ot5 = 9;
int ot6 = 10;
int ot7 = 11;
int rst = 12;
int done1 = -1;
int done2 = -1;
bool done = false;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("\n");
  pinMode(clk, OUTPUT);
  pinMode(dat, OUTPUT);
  pinMode(ot0, INPUT);
  pinMode(ot1, INPUT);
  pinMode(ot2, INPUT);
  pinMode(ot3, INPUT);
  pinMode(ot4, INPUT);
  pinMode(ot5, INPUT);
  pinMode(ot6, INPUT);
  pinMode(ot7, INPUT);
  pinMode(rst, OUTPUT);
  delay(500);
  digitalWrite(rst, LOW);
  delay(500);
  digitalWrite(rst, HIGH);
  randomSeed(analogRead(0));
  for(int i = 0; i < arrLen; i++){
    longArr[i] = random();
  }
  flipOneBit(longArr, longArrFlip);
  delay(5000);
  printArrHex(longArr);
  printArrHex(longArrFlip);
}

void loop() {
  if(done1 == -1){
    done1 = compareCRC(longArr);
    printArrHex(longArr);
    printArrHex(longArrFlip);
  }

  if(done2 == -1){
    delay(500);
    digitalWrite(rst, LOW);
    delay(500);
    digitalWrite(rst, HIGH);

    done2 = compareCRC(longArrFlip);
  }

  if(done == false){ 
    printArrHex(longArr);
    Serial.print("CRC Data Output: ");
    Serial.println(done1, HEX);
    Serial.println("");
    printArrHex(longArrFlip);
    Serial.print("CRC Data Output: ");
    Serial.println(done2, HEX);

    Serial.print("\nIf either outputs is -2 then the wires are not connected properly or the CRC circuit does not work. If the output is -1 then the function did not run.");
    done = true;
  }
}

//This function runs the code to compare the CRC to the arduino
//The long arr[] input is converted to bits and then runs it through the CRC from right to left
int compareCRC(long arr[]){
  int idx = 0;
  int bit = 0;
  int x = 0;
  int board = 0;
  uint8_t crc = 0; //https://stackoverflow.com/questions/35246212/is-there-an-actual-8-bit-integer-data-type-in-c
  while ((bit != 0) || (idx < arrLen)){
    Serial.print("Clk #: ");
    Serial.println(x);
    long number = arr[idx];
    int randBit = (number>>bit)%2;
    Serial.print("Data: ");
    if(randBit){
      crc = maximDallas(1, crc);
      digitalWrite(dat, HIGH);
      Serial.println("1");
    } else {
      crc = maximDallas(0, crc);
      digitalWrite(dat, LOW);
      Serial.println("0");
    }
    Serial.print("Number: "); 
    Serial.println(number, HEX);
    digitalWrite(clk, HIGH);
    digitalWrite(clk, LOW);
    Serial.print("Board: ");
    board = (128*digitalRead(4) + 64*digitalRead(5) + 32*digitalRead(6) + 16*digitalRead(7)
             + 8*digitalRead(8) +  4*digitalRead(9) +  2*digitalRead(10) +   digitalRead(11));
    Serial.println(board, BIN);
    Serial.print("crc:   ");
    Serial.println(crc, BIN);
    Serial.println(" ");
    if(board != crc){
      return(-2);
    }
    bit = bit + 1;
    x = x + 1;
    if(bit >= 32){
      idx = idx + 1;
      bit = 0;
    }
  }
  return(board);
}

//This function flips one bit in the array, the array must be arrLen long.
//long arr0[] is copied to long arr1[] and then one random bit is chosen to be flipped
void flipOneBit(long arr0[], long arr1[]){
  int idxRand = random(arrLen); // the long in the array to pull from
  long bitRand = random(16); // the bit to flip
  for(int i = 0; i < arrLen; i++){
    arr1[i] = arr0[i];
  }
  long bitFlip = 1 << bitRand;
  if((arr0[idxRand] >> bitRand)%2){
    arr1[idxRand] = arr0[idxRand] - bitFlip;
  }else{
    arr1[idxRand] = arr0[idxRand] + bitFlip;
  }
  Serial.print("Index Flipped: ");
  Serial.println(idxRand);
  Serial.print("Bit Flipped: ");
  Serial.println(bitRand);
}

//This is the function for running the CRC on just the arduino
//uint8t inputBit is an 1 bit variable holds the input but to run through the CRC (uint8_t is used to save memory) 
//uint8_t crc is an 8 bit variable which holds the CRC value while the bits are being input
int maximDallas(uint8_t inputBit, uint8_t crc){ //https://forum.arduino.cc/t/crc-for-dallas-1-wire/37745 (without j loop)
  uint8_t mix = (crc^inputBit)&0x01;
  crc >>= 1;
  if (mix){
    crc ^= 0x8C;
  }
  return crc;
}

//This function takes an array of longs and prints it out as a string of hex values
//long input[] holds the longs to be converted to HEX
void printArrHex(long input[]){
  Serial.println("Note: The CRC data input goes from right to left...");
  Serial.print("Data Input: ");
  for(int i = arrLen-1; i >= 0; i--){
    if(input[i] <= 268435456){
      Serial.print("0");  
      if(input[i] <= 16777216){
        Serial.print("0");  
        if(input[i] <= 1048576){
          Serial.print("0");  
          if(input[i] <= 65536){
            Serial.print("0");
            if(input[i] <= 4096){
              Serial.print("0");  
              if(input[i] <= 256){
                Serial.print("0");  
                if(input[i] <= 16){
                  Serial.print("0");  
                  if(input[i] <= 1){
                    Serial.print("0");  
                  }
                }
              }
            }  
          }
        }
      }
    }

    Serial.print(input[i], HEX);
  }
  Serial.println("");
}