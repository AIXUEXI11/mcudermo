#include "LPC11xx.h"                    // Device header
#include <stdio.h>
#include <string.h>
# define UART_BPS   9600               
char GcRcvBuf[20];
void Delay(uint32_t ulTime)
{
	uint32_t i;
	i=0;
	while(ulTime--)
	{
		for(i=0;i<5000;i++);
	}
}
void ADC_Init(void)
{
	LPC_SYSCON->SYSAHBCLKCTRL |=(1<<16);
	LPC_IOCON->R_PIO0_11 &= ~0xBF;
	LPC_IOCON->R_PIO0_11 |= 0x02;
	LPC_SYSCON->PDRUNCFG &= ~(0x01<<4);
	LPC_SYSCON->SYSAHBCLKCTRL |= (0x01<<13);
	LPC_ADC->CR = (0x01<<0) |
	              ((SystemCoreClock/1000000-1)<<8) |
	              (0<<16) |
	              (0<<17) |
	              (0<<24) |
	              (0<<27);
}
void UART_Init(void)
{ 
	
	uint16_t usFdiv;
	LPC_SYSCON->SYSAHBCLKCTRL|=(1<<16);
	LPC_IOCON->PIO1_6 &=~0x07;
	LPC_IOCON->PIO1_6 |=0x01;            
	LPC_IOCON->PIO1_7 &=~0x07;
	LPC_IOCON->PIO1_7 |=0x01;            
	LPC_SYSCON->SYSAHBCLKCTRL &=~(1<<16); 
	LPC_SYSCON->SYSAHBCLKCTRL |=(1<<12);  
	LPC_SYSCON->UARTCLKDIV = 0x01;       
	LPC_UART->LCR = 0x83;              
	usFdiv = (SystemCoreClock/LPC_SYSCON->UARTCLKDIV/16)/UART_BPS;
	LPC_UART->DLM = usFdiv/256;           
	LPC_UART->DLM = usFdiv%256;        
	LPC_UART->LCR = 0x03;                 
	LPC_UART->FCR = 0x07;                
}
void UART_SendByte(uint8_t ucDat)
{
	LPC_UART->THR=ucDat;
	while((LPC_UART->LSR&0x040)==0);
}
void UART_SendStr(char * pucStr)
{
	while(1){
		if(*pucStr == '\0')break;
		UART_SendByte(* pucStr++);
	}
}
void LED_Init()
{
	LPC_SYSCON->SYSAHBCLKCTRL |=(1<<6);
	LPC_IOCON->PIO2_6 &=0xF8;
	LPC_GPIO2->DIR |=(1<<0);
	LPC_GPIO2->DIR |=(1<<1);
	LPC_GPIO2->DIR |=(1<<2);
	LPC_GPIO2->DIR |=(1<<3);
	LPC_GPIO2->DIR |=(1<<4);
	LPC_GPIO2->DIR |=(1<<5);
	LPC_GPIO2->DIR |=(1<<6);
	LPC_GPIO2->DIR |=(1<<7);
}
int main(void)
{
	uint32_t i;
	uint32_t ulADCData;
	uint32_t ulADCBuf;
	UART_Init();
	ADC_Init();
	LED_Init();
	while(1)
	{
		ulADCData=0;
		for(i=0;i<10;i++)
		{
			LPC_ADC->CR|=(1<<24);
			while((LPC_ADC->DR[0]&0x80000000)==0);
			LPC_ADC->CR|=(1<<24);
			while((LPC_ADC->DR[0]&0x80000000)==0);
			ulADCBuf = LPC_ADC->DR[0];
			ulADCBuf = (ulADCBuf>>6)&0x3ff;
			ulADCData += ulADCBuf;
		}
		ulADCData = ulADCData/10;
		ulADCData = (ulADCData*3300)/1024;
		sprintf(GcRcvBuf,"VIN0=%4dmV\r\n",ulADCData);
		UART_SendStr(GcRcvBuf);
		Delay(200);
		if(ulADCData<400)    LPC_GPIO2->DATA = 0xFE;
		if(ulADCData>400&&ulADCData<800) LPC_GPIO2->DATA = 0xFC;
		if(ulADCData>800&&ulADCData<1200) LPC_GPIO2->DATA = 0xF8;
		if(ulADCData>1200&&ulADCData<1600)LPC_GPIO2->DATA = 0xF0;
		if(ulADCData>1600&&ulADCData<2000)LPC_GPIO2->DATA = 0xE0;
		if(ulADCData>2000&&ulADCData<2400)LPC_GPIO2->DATA = 0xC0;
		if(ulADCData>2400&&ulADCData<2800)LPC_GPIO2->DATA = 0x80;
		if(ulADCData>2800&&ulADCData<3200) LPC_GPIO2->DATA = 0x00;
	}
}
