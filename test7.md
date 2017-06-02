#include "LPC11xx.h"                    // Device header

static uint8_t count = 5;
void PIOINT3_IRQHandler()               
{
	if((LPC_GPIO3->MIS & (1<<0)) == (1<<0))
	{
		count ++;
		if(count>10) count=10;
		LPC_TMR32B1->MR1 = SystemCoreClock/1000 * count/10;
		LPC_GPIO3->IC=(1<<0);
	}
	if((LPC_GPIO3->MIS & (1<<1)) == (1<<1))
	{
		count --;
		if(count<2) count=2;
		LPC_TMR32B1->MR1 = SystemCoreClock/1000 * count/10;
		LPC_GPIO3->IC=(1<<1);
	}
}
void CT32B1_Init(uint32_t interval)
{
	LPC_SYSCON->SYSAHBCLKCTRL |=(1<<16);//´ò¿ªÊ±ÖÓIOCON
	LPC_IOCON->R_PIO1_2 &=~(0x07);
	LPC_IOCON->R_PIO1_2 |=0x03;
	LPC_SYSCON->SYSAHBCLKCTRL &=~(1<<16);
	LPC_SYSCON->SYSAHBCLKCTRL |=(1<<10);
	LPC_TMR32B1->TCR = 0x02;
	LPC_TMR32B1->PR = 0;
	LPC_TMR32B1->MCR = 0x02<<9;
	LPC_TMR32B1->PWMC = 0x02;
	LPC_TMR32B1->MR1 = interval/2;
	LPC_TMR32B1->MR3 = interval;
	LPC_TMR32B1->TCR = 0x01;
}

int main(void)
{
	CT32B1_Init(SystemCoreClock/1000);
	LPC_GPIO3->DIR &=~(1<<0);
	LPC_GPIO3->DIR &=~(1<<1);
	LPC_GPIO3->IE |=(1<<0);
	LPC_GPIO3->IE |=(1<<1);
	LPC_GPIO3->IS &=~(1<<0);
	LPC_GPIO3->IS &=~(1<<1);
	LPC_GPIO3->IEV &=~(1<<0);
	LPC_GPIO3->IEV &=~(1<<2);
		NVIC_EnableIRQ(EINT3_IRQn);

	
	while(1);
	
}
