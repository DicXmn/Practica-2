#include <18F4620.h>
#include <stdlib.h>
#include <stdio.h>
#fuses INTRC_IO, NOFCMEN, NOIESO, PUT, NOBROWNOUT, NOWDT
#fuses NOPBADEN, NOMCLR, STVREN, NOLVP, NODEBUG
#use delay(clock=16000000)
#use fast_io(A)
#use fast_io(B)
#use fast_io(C)
#use fast_io(D)
#use fast_io(E)
int selection=0;
signed long results=0;
int counter=4;
int counter2=8;
int counter3=32;
int flag=0;

void ringCounter(void);

void main(void)
{
   setup_oscillator(OSC_16MHZ);
   set_tris_A(0xC0);
   set_tris_B(0xF0);
   set_tris_C(0xFF);
   set_tris_D(0xFF);
   set_tris_E(0x08); 
   while(true) 
   {   
      if(input(pin_B7)==0||selection==1) 
      {
         selection=1;
         results=(long)input_d()+(long)input_c();
      }
      if(input(pin_B6)==0||selection==2) 
      {
         selection=2;
         results=(long)input_d()-(long)input_c();    
      }
      if(input(pin_B5)==0||selection==3)
      {
         selection=3;
         results=(long)input_d()*(long)input_c();      
      }   
      if(input(pin_B4)==0||selection==4)
      {
         selection=4;
         if(input_c()!=0)
            results=(long)input_d()/(long)input_c();
         else
            ringCounter();
      } 
      output_A(results);
      output_B(results>>6);
      output_E(results>>10);   
   }  
}

void ringCounter(void)
{
   if(input_c()!=0)
   {  
      results=(long)input_d()/(long)input_c();
   }
   else if(input_c()==0)
   {
      results=0;
      if(counter>0)
      {
         output_E(counter);
         counter=counter >> 1;
         delay_ms(500);
         if(counter==0 && flag==0)
            counter2=8;
      }      
      else if(counter2>0)
      {  
         flag=1;
         output_E(counter);
         output_B(counter2);
         counter2 >>= 1;
         delay_ms(500);
         if(counter2==0&&flag==1)
            counter3=32;  
      }
      else
      {
         flag=2;
         output_E(counter);
         output_B(counter2);
         output_A(counter3);
         counter3 >>=1;
         delay_ms(500);
         if(counter3==0&&flag==2)
         {
            counter=4;
            flag=0;         
         }         
      }
   }   
}
