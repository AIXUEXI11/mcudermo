#include "LPC11xx.h"                    // Device header
#define UART_BPS 115200
uint8_t ucRcvData = 0xFF;
void UART_Init(void)
{
	uint16_t usFdiv;
	LPC_SYSCON->SYSAHBCLKCTRL |= (1<<16); //IOCONÊ±ÖÓ
	LPC_IOCON->PIO1_6 &=~0x07;
	LPC_IOCON->PIO1_6 |=0x01;             //1.6½ÅÉèÎªRXD
	LPC_IOCON->PIO1_7 &=~0x07;
	LPC_IOCON->PIO1_7 |=0x01;             //1.7½ÅÉèÎªTXD
	LPC_SYSCON->SYSAHBCLKCTRL &=~(1<<16); //¹Ø±ÕÊ±ÖÓ
	LPC_SYSCON->SYSAHBCLKCTRL |=(1<<12);  //UARTÊ±ÖÓ¿ªÆô
	LPC_SYSCON->UARTCLKDIV = 0x01;        //Ê±ÖÓ·ÖÆµ1
	LPC_UART->LCR = 0x83;                 //8Î»´«Êä,1Í£Ö¹£¬ÎÞÆæÅ¼Ð£Ñé£¬ÔÊÐí·ÃÎÊ³ýÊýËø´æÆ÷
	usFdiv = (SystemCoreClock/LPC_SYSCON->UARTCLKDIV/16)/UART_BPS;
	LPC_UART->DLM = usFdiv/256;           //Ð´Ëø´æÆ÷×î¸ßÎ»
	LPC_UART->DLM = usFdiv%256;           //×îµÍÎ»
	LPC_UART->LCR = 0x03;                 //DLABÖÃ0
	LPC_UART->FCR = 0x07;                 //FIFO
}
void delay(void)
{
	uint16_t i=4000;
	while(i--);
}
void LED_ON(ucBuf)
{
		LPC_GPIO2->DATA &=~(0xff);
		LPC_GPIO2->DATA |= ucBuf;
		delay();
		LPC_GPIO2->DATA = 0xFF;
}
uint8_t UART_GetByte(void)
{

	while((LPC_UART->LSR&0x01)==0)
			LED_ON(ucRcvData);
	ucRcvData=LPC_UART->RBR;
	return(ucRcvData);
}
void UART_SendByte(uint8_t ucDat)
{
	LPC_UART->THR=ucDat;
	while((LPC_UART->LSR&0x040)==0);
}
void LEDInit(void)
{
	LPC_GPIO2->DIR = 0xFF;
}


int main(void)
{
	uint8_t ucBuf;
	UART_Init();
	LEDInit();
	while(1)
	{
		ucBuf=UART_GetByte();
		UART_SendByte(ucBuf);
	}
}
