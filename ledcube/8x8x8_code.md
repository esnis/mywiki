## 8x8x8 led并行接口arduino代码
````
#include 
#include 
#include 
#define AXIS_X 1
#define AXIS_Y 2
#define AXIS_Z 3
int CUBE_SIZE = 8;

volatile unsigned char cube[8][8];
volatile int current_layer = 0;

void setup()
{
  int i;
  
  for(i=0; i<14; i++)
    pinMode(i, OUTPUT);
  
  // pinMode(A0, OUTPUT) as specified in the arduino reference didn't work. So I accessed the registers directly.
  DDRC = 0xff;
  PORTC = 0x00;
  
  // Reset any PWM configuration that the arduino may have set up automagically!
  TCCR2A = 0x00;
  TCCR2B = 0x00;

  TCCR2A |= (0x01 << WGM21); // CTC mode. clear counter on TCNT2 == OCR2A
  OCR2A = 10; // Interrupt every 25600th cpu cycle (256*100)
  TCNT2 = 0x00; // start counting at 0
  TCCR2B |= (0x01 << CS22) | (0x01 << CS21); // Start the clock with a 256 prescaler
  
  TIMSK2 |= (0x01 << OCIE2A);
}

ISR (TIMER2_COMPA_vect)
{
  int i;
  
  // all layer selects off
  PORTC = 0x00;
  PORTB &= 0x0f;
  
  PORTB |= 0x08; // output enable off.
  
  for (i=0; i<8; i++)
  {
    PORTD = cube[current_layer][i];
    PORTB = (PORTB & 0xF8) | (0x07 & (i+1));
  }
  
  PORTB &= 0b00110111; // Output enable on.
  
  if (current_layer < 6)
  {
    PORTC = (0x01 << current_layer);
  } else if (current_layer == 6)
  {
    digitalWrite(12, HIGH);
  } else
  {
    digitalWrite(13, HIGH);
  }
  
  current_layer++;
  
  if (current_layer == 8)
    current_layer = 0;
}

void loop()
{
  int i,x,y,z,m,n,d;
  d=100;
   for (y=0; y<8; y++){
    for (x=0; x<8; x++){
      for (z=0; z<8; z++){

        setvoxel(x,y,z); 
      }

        delay (d);
      
        for (z=0; z<8; z++){
          clrvoxel(x,y,z); 
        }
      }
   }
     for (z=0; z<8; z++){
       for (y=0; y<8; y++){
          for (x=0; x<8; x++){
        
          setvoxel(x,y,z);
           delay (d/8);
        }
      }
     }

   
   fill (0X00);
  
  
  while (true)
  {
    effect_intro();
  effect_wormsqueeze (2, AXIS_Z, -1, 100, 1000);
  zoom_pyramid();
  zoom_pyramid_clear();
 
    effect_text(1200,20,34);
    sinelines(4000,10);
    linespin(1500,10);
    

    effect_planboing(AXIS_Z, 900);
    effect_planboing(AXIS_Y, 900);
    effect_planboing(AXIS_X, 900);
    effect_text(1200,0,21);
    effect_blinky2();
    mirror_ripples(600,400);
    effect_axis_updown_randsuspend(AXIS_Z, 550,5000,0);
effect_axis_updown_randsuspend(AXIS_Z, 550,5000,1);
effect_axis_updown_randsuspend(AXIS_Z, 550,5000,0);
effect_axis_updown_randsuspend(AXIS_Z, 550,5000,1);
effect_axis_updown_randsuspend(AXIS_X, 550,5000,0);
effect_axis_updown_randsuspend(AXIS_X, 550,5000,1);
effect_axis_updown_randsuspend(AXIS_Y, 550,5000,0);
effect_axis_updown_randsuspend(AXIS_Y, 550,5000,1);
effect_rand_patharound(200,500);
    fireworks(10,30,500);

    effect_random_filler(75,1);
    effect_random_filler(75,0);
     zoom_pyramid();
  zoom_pyramid_clear();
    effect_rain(100);
    side_ripples (300, 500);
    effect_text_up(1200,0,21);
    effect_random_sparkle();
    quad_ripples (600,300);
    effect_boxside_randsend_parallel (AXIS_X, 0, 150, 1);
    effect_boxside_randsend_parallel (AXIS_X, 1, 150, 1);
    effect_boxside_randsend_parallel (AXIS_Y, 0, 150, 1);
    effect_boxside_randsend_parallel (AXIS_Y, 1, 150, 1);
    effect_boxside_randsend_parallel (AXIS_Z, 0, 150, 1);
    effect_boxside_randsend_parallel (AXIS_Z, 1, 150, 1);
    
  }
}


// ==========================================================================================
//   Effect functions
// ==========================================================================================

void fireworks (int iterations, int n, int delay)
{
        fill(0x00);

        int i,f,e;

        float origin_x = 3;
        float origin_y = 3;
        float origin_z = 3;

        int rand_y, rand_x, rand_z;

        float slowrate, gravity;

        // Particles and their position, x,y,z and their movement, dx, dy, dz
        float particles[n][6];

        for (i=0; i<25; e++)
                {
                        slowrate = 1+tan((e+0.1)/20)*10;
                        
                        gravity = tan((e+0.1)/20)/2;

                        for (f=0; f<4;x++)
                        for(y=0;y<4;y++)
                        {
                                // x+y*4 gives no. from 0-15 for sqrt_LUT
                                distance=sqrt_LUT[x+y*4];// distance is 0-50 roughly
                                // height is sin of distance + iteration*4
                                //height=4+totty_sin(LUT,distance+i)/52;
                                height=(196+totty_sin(LUT,distance+i))/49;
                                // Use 4-way mirroring to save on calculations
                                setvoxel(x,y,height);
                                setvoxel(7-x,y,height);
                                setvoxel(x,7-y,height);
                                setvoxel(7-x,7-y,height);
                                setvoxel(x,y,7-height);
                                setvoxel(7-x,y,7-height);
                                setvoxel(x,7-y,7-height);
                                setvoxel(7-x,7-y,7-height);
                                setvoxel(x,height,y);
                                setvoxel(7-x,height,y);
                                setvoxel(x,height,7-y);
                                setvoxel(7-x,height,7-y);
                                setvoxel(x,7-height,y);
                                setvoxel(7-x,7-height,y);
                                setvoxel(x,7-height,7-y);
                                setvoxel(7-x,7-height,7-y);


                        }
                delay_ms(delay);
        }
}



// **********************************************************

void effect_random_sparkle_flash (int iterations, int voxels, int delay)
{
        int i;
        int v;
        for (i = 0; i < iterations; i++)
        {
                for (v=0;v<=voxels;v++)
                        setvoxel(rand()%8,rand()%8,rand()%8);
                        
                delay_ms(delay);
                fill(0x00);
        }
}

// blink 1 random voxel, blink 2 random voxels..... blink 20 random voxels
// and back to 1 again.
void effect_random_sparkle (void)
{
        int i;
        
        for (i=1;i<20;i++)
        {
                effect_random_sparkle_flash(5,i,200);
        }
        
        for (i=20;i>=1;i--)
        {
                effect_random_sparkle_flash(5,i,200);
        }
        
}

int effect_telcstairs_do(int x, int val, int delay)
{
        int y,z;

        for(y = 0, z = x; y <= z; y++, x--)
        {
                if(x < CUBE_SIZE && y < CUBE_SIZE)
                {
                        cube[x][y] = val;
                }
        }
        delay_ms(delay);
        return z;
}

void effect_telcstairs (int invert, int delay, int val)
{
        int x;

        if(invert)
        {
                for(x = CUBE_SIZE*2; x >= 0; x--)
                {
                        x = effect_telcstairs_do(x,val,delay);
                }
        }
        else
        {
                for(x = 0; x < CUBE_SIZE*2; x++)
                {
                        x = effect_telcstairs_do(x,val,delay);
                }
        }
}



void draw_positions_axis (char axis, unsigned char positions[64], int invert)
{
        int x, y, p;
        
        fill(0x00);
        
        for (x=0; x<8; x++)
        {
                for (y=0; y<8; y++)
                {
                        if (invert)
                        {
                                p = (7-positions[(x*8)+y]);
                        } else
                        {
                                p = positions[(x*8)+y];
                        }
                
                        if (axis == AXIS_Z)
                                setvoxel(x,y,p);
                                
                        if (axis == AXIS_Y)
                                setvoxel(x,p,y);
                                
                        if (axis == AXIS_X)
                                setvoxel(p,y,x);
                }
        }
        
}

void effect_wormsqueeze (int size, int axis, int direction, int iterations, int delay)
{
        int x, y, i,j,k, dx, dy;
        int cube_size;
        int origin = 0;
        
        if (direction == -1)
                origin = 7;
        
        cube_size = 8-(size-1);
        
        x = rand()%cube_size;
        y = rand()%cube_size;
        
        for (i=0; i 0 && (x+dx) < cube_size)
                        x += dx;
                        
                if ((y+dy) > 0 && (y+dy) < cube_size)
                        y += dy;
        
                shift(axis, direction);
                

                for (j=0; j<8 ;x++)
                {
                        x_dividor = 2 + sin((float)i/100)+1;
                        ripple_height = 3 + (sin((float)i/200)+1)*6;

                        sine_base = (float) i/40 + (float) x/x_dividor;

                        left = 4 + sin(sine_base)*ripple_height;
                        right = 4 + cos(sine_base)*ripple_height;
                        right = 7-left;

                        //printf("%i %i \n", (int) left, (int) right);

                        line_3d(0-3, x, (int) left, 7+3, x, (int) right);
                        //line_3d((int) right, 7, x);
                }
        
        // delay_ms(delay);
        fill(0x00);
        }
}


void effect_boxside_randsend_parallel (char axis, int origin, int delay, int mode)
{
        int i;
        int done;
        unsigned char cubepos[64];
        unsigned char pos[64];
        int notdone = 1;
        int notdone2 = 1;
        int sent = 0;
        
        for (i=0;i<64;i++)
        {
                pos[i] = 0;
        }
        
        while (notdone)
        {
                if (mode == 1)
                {
                        notdone2 = 1;
                        while (notdone2 && sent<64)
                        {
                                i = rand()d;
                                if (pos[i] == 0)
                                {
                                        sent++;
                                        pos[i] += 1;
                                        notdone2 = 0;
                                }
                        }
                } else if (mode == 2)
                {
                        if (sent<64)
                        {
                                pos[sent] += 1;
                                sent++;
                        }
                }
                
                done = 0;
                for (i=0;i<64;i++)
                {
                        if (pos[i] > 0 && pos[i] <7)
                        {
                                pos[i] += 1;
                        }
                                
                        if (pos[i] == 7)
                                done++;
                }
                
                if (done == 64)
                        notdone = 0;
                
                for (i=0;i<64;i++)
                {
                        if (origin == 0)
                        {
                                cubepos[i] = pos[i];
                        } else
                        {
                                cubepos[i] = (7-pos[i]);
                        }
                }
                
                
                delay_ms(delay);
                draw_positions_axis(axis,cubepos,0);

        }
        
}


void effect_rain (int iterations)
{
        int i, ii;
        int rnd_x;
        int rnd_y;
        int rnd_num;
        
        for (ii=0;ii< rnd_num;i++)
                {
                        rnd_x = rand()%8;
                        rnd_y = rand()%8;
                        setvoxel(rnd_x,rnd_y,7);
                }
                
                delay_ms(1000);
                shift(AXIS_Z,-1);
        }
}

// Set or clear exactly 512 voxels in a random order.
void effect_random_filler (int delay, int state)
{
        int x,y,z;
        int loop = 0;
        
        
        if (state == 1)
        {
                fill(0x00);
        } else
        {
                fill(0xff);
        }
        
        while (loop<511)
        {
                x = rand()%8;
                y = rand()%8;
                z = rand()%8;

                if ((state == 0 && getvoxel(x,y,z) == 0x01) || (state == 1 && getvoxel(x,y,z) == 0x00))
                {
                        altervoxel(x,y,z,state);
                        delay_ms(delay);
                        loop++;
                }       
        }
}

void effect_axis_updown_randsuspend (char axis, int delay, int sleep, int invert)
{
        unsigned char positions[64];
        unsigned char destinations[64];

        int i,px;
        
    // Set 64 random positions
        for (i=0; i<64; i++)
        {
                positions[i] = 0; // Set all starting positions to 0
                destinations[i] = rand()%8;
        }

    // Loop 8 times to allow destination 7 to reach all the way
        for (i=0; i<8; i++)
        {
        // For every iteration, move all position one step closer to their destination
                for (px=0; px<64; px++)
                {
                        if (positions[px]<64; i++)
        {
                destinations[i] = 7;
        }
        
    // Suspend the positions in mid-air for a while
        delay_ms(sleep);
        
    // Then do the same thing one more time
        for (i=0; i<8; i++)
        {
                for (px=0; px<64; px++)
                {
                        if (positions[px]destinations[px])
                        {
                                positions[px]--;
                        }
                }
                draw_positions_axis (axis, positions,invert);
                delay_ms(delay);
        }
}



void effect_blinky2()
{
        int i,r;
        fill(0x00);
        
        for (r=0;r<2;r++)
        {
                i = 750;
                while (i>0)
                {
                        fill(0x00);
                        delay_ms(i);
                        
                        fill(0xff);
                        delay_ms(100);
                        
                        i = i - (15+(1000/(i/10)));
                }
                
                delay_ms(1000);
                
                i = 750;
                while (i>0)
                {
                        fill(0x00);
                        delay_ms(751-i);
                        
                        fill(0xff);
                        delay_ms(100);
                        
                        i = i - (15+(1000/(i/10)));
                }
        }

}

// Draw a plane on one axis and send it back and forth once.
void effect_planboing (int plane, int speed)
{
        int i;
        for (i=0;i<8;i++)
        {
                fill(0x00);
        setplane(plane, i);
                delay_ms(speed);
        }
        
        for (i=7;i>=0;i--)
        {
                fill(0x00);
        setplane(plane,i);
                delay_ms(speed);
        }
}


/
      "ror %[out] \n\t"  
      "lsl __tmp_reg__  \n\t"   
      "ror %[out] \n\t"
      "lsl __tmp_reg__  \n\t"   
      "ror %[out] \n\t"
      "lsl __tmp_reg__  \n\t"   
      "ror %[out] \n\t"
     
      "lsl __tmp_reg__  \n\t"   
      "ror %[out] \n\t"
      "lsl __tmp_reg__  \n\t"   
      "ror %[out] \n\t"
      "lsl __tmp_reg__  \n\t"   
      "ror %[out] \n\t"
      "lsl __tmp_reg__  \n\t"   
      "ror %[out] \n\t"
      
      : [out] "=r" (result) : [in] "r" (x));
      return(result);
}
 
/

// **********************************************************

volatile const unsigned char font[910] [5] = {
//volatile const unsigned char font[455] EEMEM = {
        0x00,0x00,0x00,0x00,0x00,0x00,0x5f,0x5f,0x00,0x00,      //  !
        0x00,0x03,0x00,0x03,0x00,0x14,0x7f,0x14,0x7f,0x14,      // "#
        0x24,0x2a,0x7f,0x2a,0x12,0x23,0x13,0x08,0x64,0x62,      // $%
        0x36,0x49,0x55,0x22,0x50,0x00,0x05,0x03,0x00,0x00,      // &'
        0x00,0x1c,0x22,0x41,0x00,0x00,0x41,0x22,0x1c,0x00,      // ()
        0x14,0x08,0x3e,0x08,0x14,0x08,0x08,0x3e,0x08,0x08,      // *+
        0x00,0x50,0x30,0x00,0x00,0x08,0x08,0x08,0x08,0x08,      // ,-
        0x00,0x60,0x60,0x00,0x00,0x20,0x10,0x08,0x04,0x02,      // ./
        0x3e,0x51,0x49,0x45,0x3e,0x00,0x42,0x7f,0x40,0x00,      // 01
        0x42,0x61,0x51,0x49,0x46,0x21,0x41,0x45,0x4b,0x31,      // 23
        0x18,0x14,0x12,0x7f,0x10,0x27,0x45,0x45,0x45,0x39,      // 45
        0x3c,0x4a,0x49,0x49,0x30,0x01,0x71,0x09,0x05,0x03,      // 67
        0x36,0x49,0x49,0x49,0x36,0x06,0x49,0x49,0x29,0x1e,      // 89
        0x00,0x36,0x36,0x00,0x00,0x00,0x56,0x36,0x00,0x00,      // :;
        0x08,0x14,0x22,0x41,0x00,0x14,0x14,0x14,0x14,0x14,      // <=
        0x00,0x41,0x22,0x14,0x08,0x02,0x01,0x51,0x09,0x06,      // >?
        0x32,0x49,0x79,0x41,0x3e,0x7e,0x11,0x11,0x11,0x7e,      // @A
        0x7f,0x49,0x49,0x49,0x36,0x3e,0x41,0x41,0x41,0x22,      // BC
        0x7f,0x41,0x41,0x22,0x1c,0x7f,0x49,0x49,0x49,0x41,      // DE
        0x7f,0x09,0x09,0x09,0x01,0x3e,0x41,0x49,0x49,0x7a,      // FG
        0x7f,0x08,0x08,0x08,0x7f,0x00,0x41,0x7f,0x41,0x00,      // HI
        0x20,0x40,0x41,0x3f,0x01,0x7f,0x08,0x14,0x22,0x41,      // JK
        0x7f,0x40,0x40,0x40,0x40,0x7f,0x02,0x0c,0x02,0x7f,      // LM
        0x7f,0x04,0x08,0x10,0x7f,0x3e,0x41,0x41,0x41,0x3e,      // NO
        0x7f,0x09,0x09,0x09,0x06,0x3e,0x41,0x51,0x21,0x5e,      // PQ
        0x7f,0x09,0x19,0x29,0x46,0x46,0x49,0x49,0x49,0x31,      // RS
        0x01,0x01,0x7f,0x01,0x01,0x3f,0x40,0x40,0x40,0x3f,      // TU
        0x1f,0x20,0x40,0x20,0x1f,0x3f,0x40,0x38,0x40,0x3f,      // VW
        0x63,0x14,0x08,0x14,0x63,0x07,0x08,0x70,0x08,0x07,      // XY
        0x61,0x51,0x49,0x45,0x43,0x00,0x7f,0x41,0x41,0x00,      // Z[
        0x02,0x04,0x08,0x10,0x20,0x00,0x41,0x41,0x7f,0x00,      // \]
        0x04,0x02,0x01,0x02,0x04,0x40,0x40,0x40,0x40,0x40,      // ^_
        0x00,0x01,0x02,0x04,0x00,0x20,0x54,0x54,0x54,0x78,      // `a
        0x7f,0x48,0x44,0x44,0x38,0x38,0x44,0x44,0x44,0x20,      // bc
        0x38,0x44,0x44,0x48,0x7f,0x38,0x54,0x54,0x54,0x18,      // de
        0x08,0x7e,0x09,0x01,0x02,0x0c,0x52,0x52,0x52,0x3e,      // fg
        0x7f,0x08,0x04,0x04,0x78,0x00,0x44,0x7d,0x40,0x00,      // hi
        0x20,0x40,0x44,0x3d,0x00,0x7f,0x10,0x28,0x44,0x00,      // jk
        0x00,0x41,0x7f,0x40,0x00,0x7c,0x04,0x18,0x04,0x78,      // lm
        0x7c,0x08,0x04,0x04,0x78,0x38,0x44,0x44,0x44,0x38,      // no
        0x7c,0x14,0x14,0x14,0x08,0x08,0x14,0x14,0x18,0x7c,      // pq
        0x7c,0x08,0x04,0x04,0x08,0x48,0x54,0x54,0x54,0x20,      // rs
        0x04,0x3f,0x44,0x40,0x20,0x3c,0x40,0x40,0x20,0x7c,      // tu
        0x1c,0x20,0x40,0x20,0x1c,0x3c,0x40,0x30,0x40,0x3c,      // vw
        0x44,0x28,0x10,0x28,0x44,0x0c,0x50,0x50,0x50,0x3c,      // xy
        0x44,0x64,0x54,0x4c,0x44                                // z
};

volatile const unsigned char bitmaps[13][8] = {
// volatile const unsigned char bitmaps[13][8] EEMEM = {
        {0xc3,0xc3,0x00,0x18,0x18,0x81,0xff,0x7e}, // smiley 3 small
        {0x3c,0x42,0x81,0x81,0xc3,0x24,0xa5,0xe7}, // Omega
        {0x00,0x04,0x06,0xff,0xff,0x06,0x04,0x00},  // Arrow
        {0x81,0x42,0x24,0x18,0x18,0x24,0x42,0x81}, // X
        {0xBD,0xA1,0xA1,0xB9,0xA1,0xA1,0xA1,0x00}, // ifi
        {0xEF,0x48,0x4B,0x49,0x4F,0x00,0x00,0x00}, // TG
        {0x38,0x7f,0xE6,0xC0,0xE6,0x7f,0x38,0x00}, // Commodore symbol
        {0x00,0x22,0x77,0x7f,0x3e,0x3e,0x1c,0x08}, // Heart
        {0x1C,0x22,0x55,0x49,0x5d,0x22,0x1c,0x00}, // face
        {0x37,0x42,0x22,0x12,0x62,0x00,0x7f,0x00}, // ST
        {0x89,0x4A,0x2c,0xF8,0x1F,0x34,0x52,0x91}, // STAR
        {0x18,0x3c,0x7e,0xdb,0xff,0x24,0x5a,0xa5}, // Space Invader
        {0x00,0x9c,0xa2,0xc5,0xc1,0xa2,0x9c,0x00}       // Fish
};

const unsigned char paths[44] PROGMEM = {0x07,0x06,0x05,0x04,0x03,0x02,0x01,0x00,0x10,0x20,0x30,0x40,0x50,0x60,0x70,0x71,0x72,0x73,0x74,0x75,0x76,0x77,0x67,0x57,0x47,0x37,0x27,0x17,
0x04,0x03,0x12,0x21,0x30,0x40,0x51,0x62,0x73,0x74,0x65,0x56,0x47,0x37,0x26,0x15}; // circle, len 16, offset 28

void font_getpath (unsigned char path, unsigned char *destination, int length)
{
        int i;
        int offset = 0;
        
        if (path == 1)
                offset=28;
        
        for (i = 0; i < length; i++)
                destination[i] = pgm_read_byte(&paths[i+offset]);
}


// ************************************************************

void effect_pathmove (unsigned char *path, int length)
{
        int i,z;
        unsigned char state;
        
        for (i=(length-1);i>=1;i--)
        {
                for (z=0;z<8;z++)
                {
                
                        state = getvoxel(((path[(i-1)]>>4) & 0x0f), (path[(i-1)] & 0x0f), z);
                        altervoxel(((path[i]>>4) & 0x0f), (path[i] & 0x0f), z, state);
                }
        }
        for (i=0;i<8;i++)
                clrvoxel(((path[0]>>4) & 0x0f), (path[0] & 0x0f),i);
}

void effect_rand_patharound (int iterations, int delay)
{
        int z, dz, i;
        z = 4;
        unsigned char path[28];
        
        font_getpath(0,path,28);
        
        for (i = 0; i < iterations; i++)
        {
                dz = ((rand()%3)-1);
                z += dz;
                
                if (z>7)
                        z = 7;
                        
                if (z<0)
                        z = 0;
                
                effect_pathmove(path, 28);
                setvoxel(0,7,z);
                delay_ms(delay);
        }
}
