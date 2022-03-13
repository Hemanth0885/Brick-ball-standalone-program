# Brick-ball-standalone-program
#include <stdio.h>
#include <conio.h>
#include <process.h>
#include <dos.h>
#include <stdlib.h>
#include <graphics.h>
#include <ctype.h>

#define NULL 0
#define YES 1
#define NO 0

int maxx, maxy, midx, midy;
int bri[5][20];

void music(int);
char mainscreen();
void drawbrick(int, int);
void erasebrick(int, int);
void bricks(void);


int main() {
 union REGS ii, oo;
 int ballx, bally, paddlex;
 int paddley, dx = 1, dy = -1, oldx, oldy;
 int gm = CGAHI, gd = CGA, playerlevel;
 int i, flag = 0, speed = 25;
 int welldone = NO, score = 0, chance = 4, area;
 int layer[5] = {
    10,
    20,
    30,
    40,
    50
 }, limit = 50, currentlayer = 4;
 char* p1, *p2;

 // initialise the graphics system 
 initgraph(&gd, &gm, "C:\\TURBOC3\\BGI");

 // get the maximum x and y screen coordinates 
 maxx = getmaxx();
 maxy = getmaxy();

 // calculate center of screen 
 midx = maxx / 2;
 midy = maxy / 2;

 // display opening screen and receive player's level 
 playerlevel = mainscreen();

 // set speed of ball as per the level chosen 
 switch (playerlevel) {
 case 'A':
 case 'a':
  speed = 15;
  break;

 case 'E':
 case 'e':
  speed = 10;
 }

 // draw the bricks, the paddle and the ball 
 rectangle(0, 0, maxx, maxy - 12);
 bricks();
 rectangle(midx - 25, maxy - 7 - 12, midx + 25, maxy - 12);
 floodfill(midx, maxy - 1 - 12, 1);
 circle(midx, maxy - 13 - 12, 12);
 floodfill(midx, maxy - 10 - 12, 1);

 // allocate memory for storing the image of the paddle 
 area = imagesize(midx - 12, maxy - 18, midx + 12, maxy - 8);
 p1 = (char*)malloc(area);

 // allocate memory for storing the image of the ball 
 area = imagesize(midx - 25, maxy
[3/12, 10:24 PM] Sujay (Adk): - 7, midx + 25, maxy - 1);
 p2 = (char*)malloc(area);

 // if memory allocation unsuccessful 
 if (p1 == NULL || p2 == NULL) {
  puts("Insufficient memory!!");
  exit(1);
 }

 // store the image of the paddle 
 // and the ball into allocated memory 
 getimage(midx - 12, maxy - 7 - 12 - 12 + 1, midx + 12, maxy - 8 - 12, p1);
 getimage(midx - 25, maxy - 7 - 12, midx + 25, maxy - 1 - 12, p2);

 // store current position of the paddle and ball 
 paddlex = midx - 25;
 paddley = maxy - 7 - 12;
 ballx = midx - 12;
 bally = maxy - 7 - 12 + 1 - 12;

 // display balls in hand ( initially 3 ) 
 gotoxy(45, 25);
 printf("Balls Remaining: ");
 for (i = 0; i < 3; i++) {
  circle(515 + i * 35, maxy - 5, 12);
  floodfill(515 + i * 35, maxy - 5, 1);
 } // display initial score 
 gotoxy(1, 25);
 // select font and alignment for displaying text 
 printf("Your Score: %4d", score);
 settextjustify(CENTER_TEXT, CENTER_TEXT);
 settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 4);
 while (1) {
  // save the current x and y coordinates of the ball 
  flag = 0;
  oldx = ballx;
  // update ballx and bally to move the ball in appropriate direction 
  oldy = bally;
  ballx = ballx + dx;
  // as per the position of ball determine the layer of bricks to check 
  bally = bally + dy;
  if (bally > 40) {
   limit = 50;
   currentlayer = 4;
  }
  else {
   if (bally > 30) {
    limit = 40;
    currentlayer = 3;
   }
   else {
    if (bally > 20) {
     limit = 30;
     currentlayer = 2;
    }
    else {
     if (bally > 10) {
      limit = 20;
      currentlayer = 1;
     }
     else {
      limit = 10;
      currentlayer = 0;
     }
    }
   }
  }

  // if the ball hits the left boundary, deflect it to the right 
  if (ballx < 1) {
   music(5);
   ballx = 1;
   dx = -dx;
[3/12, 10:25 PM] Sujay (Adk): // if the ball hits the top boundary, deflect it down 
  if (bally < 1) {
   music(5);
   bally = 1;
   dy = -dy;
  } // if the ball is in the area occupied by the bricks 
  if (bally < limit) {
   // if there is no brick present exactly at the top of the ball 
   if (bri[currentlayer][(ballx + 10) / 32] == 1) {
    // determine if the boundary of the ball touches a brick 
    for (i = 1; i <= 6; i++) {
     // check whether there is a brick to the right of the ball 
     if (bri[currentlayer][(ballx + i + 10) / 32] == 0) {
      // if there is a brick 
      ballx = ballx + i;
      flag = 1;
      break;
     } // check whether there is a brick to the left of the ball 
     if (bri[currentlayer][(ballx - i + 10) / 32] == 0) {
      ballx = ballx - i;
      flag = 1;
      break;
     }
    } // if the ball does not touch a brick at the top, left or right 
    if (!flag) {
     // check if the ball has moved above the current layer 
     if (bally < layer[currentlayer - 1]) {
      // if so, change current layer appropriately 
      currentlayer--;
      limit = layer[currentlayer];
     } // put the image of the ball at the old coordinates 
     // erase the image at the old coordinates 
     putimage(oldx, oldy, p1, OR_PUT);
     // place the image of the ball at the new coordinates 
     putimage(oldx, oldy, p1, XOR_PUT);
     // introduce delay 
     putimage(ballx, bally, p1, XOR_PUT);
     delay(speed); // carry on with moving the ball 
     continue;
    }
   } // control comes to this point only if the ball is touching a brick 
   music(4); // play music  // erase the brick hit by the ball 
   // if the brick hit happens to be on the extreme right 
   erasebrick((ballx + 10) / 32, currentlayer);
   // redraw right boundary  // if the brick hit happens to be on the extreme left 
   if ((ballx + 10) / 32 == 19) line(maxx, 0, maxx, 50);
   // redraw left boundary  // if the brick hit happens to be in the topmost layer 
   if ((ballx + 10) / 32 == 0) line(0, 0, 0, 50);
   // redraw top boundary set appropriate array
   element to 1 to indicate absence of brick 
   if (currentlayer == 0) line(0, 0, maxx, 0);
   bri[currentlayer][(ballx + 10) / 32] = 1;
   bally = bally + 1; // update the y coordinate 
   dy = -dy; // change the direction of the ball 
   score += 5; // increment score 
   gotoxy(16, 25);
   // print latest score
   if the first brick is hit during a throw 
   printf("%4d", score);

   if (welldone == NO) welldone = YES;
   else {
