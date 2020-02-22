void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_ETH_Init(void);
static void MX_USART3_UART_Init(void);
static void MX_USB_OTG_FS_PCD_Init(void);

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_ETH_Init();
  MX_USART3_UART_Init();
  MX_USB_OTG_FS_PCD_Init();
  int ContPulsos = 0, UPush = 0, DPush = 0, Mode = 0;
  int PushTime = 0, Transicion = 0, Contado = 0, Time1seg = HAL_GetTick(), Max = 0;
  ActivarIO();
  while (1)
  {
	  	  if(GPIOC->IDR == (1<<13)){//boton para cambiar el tipo de conteo
	  		if(Transicion == 0){//evitamos debounce
	  			if(Mode < 2){//chequeamos si no estamos en el ultimo modo
	  				Mode += 1;//cabiamos modo y
	  				ContPulsos = 0;//reinicioamos conteo
	  			}else{//sino reiniciamos modo 
	  				Mode = 0;
	  				ContPulsos = 0;
	  			}
	  			Transicion = 1;
	  		}
	   	    }else if(GPIOC->IDR == (0<<13) && Transicion == 1){
	   		Transicion = 0;
	   	  }
	  	  if(Mode == 2){//conteo modo binario
	  		GPIOB->ODR |=(1<<14);//encendemos led rojo para indicar el modo visualmente
	  		Max = 31;
	  	  }else{
	  		GPIOB->ODR &= ~(1<<14);
	  	  }
	  	  if(Mode == 1){//conteo modo LED grupal
	  		GPIOB->ODR |=(1<<7);//encendemos Blue rojo para indicar el modo visualmente
	  		Max=5;
	   	  }else{
	  		GPIOB->ODR &= ~(1<<7);
	   	  }
	  	  if(Mode == 0){//encendemos led verde para indicar el modo visualmente
	  		GPIOB->ODR |=(1<<0);
	  		Max=4;
	   	  }else{
	  		GPIOB->ODR &= ~(1<<0);
	  	  }
        //Funcion para LED 1 seg oscilando
	  	  if(Time1seg + 500 < HAL_GetTick()){//si estamos menor a 500ms prendemos led
	  		GPIOF->ODR |=(1<<14);
	  	  }else{//sino apagamos
	  		GPIOF->ODR &= (~(1<<14));
	  	  }
	  	  if((Time1seg + 1000) <= HAL_GetTick()){//mas de un segundo tomamos tiempo como en la ini y volvemos de nuevo
	  		Time1seg = HAL_GetTick();
	  	  }
	  	  if(GPIOC->IDR == (1<<6)){//si se recieve un toque actualizamos
	  		  //una posible conteo up
	  		  UPush = 1;
	  		  PushTime = HAL_GetTick();
	  	  }else if(GPIOC->IDR == (1<<10)){//posible down
	  		  DPush = 1;
	  		  PushTime = HAL_GetTick();
	  	  }else{
	  		UPush = 0;
	  		DPush = 0;
	  	  }
	  	  if(!UPush && !DPush && PushTime + 75 < HAL_GetTick()){ // borramos  solamente
	  		  // si pasa un tiempo desde que se solto
	  		  Contado = 0;
	  	  }
	  	  if((UPush && !DPush || !UPush && DPush) && Contado==0){//si esta un boton presionada contamos
	  		  ContarPulso(&ContPulsos, UPush, DPush, Max);
	  		  Contado = 1;
	  	  }
	  	ActualizarLEDs(Mode, ContPulsos);//despues de tener un push de botones actualizamos LEDs
  }
}

void ActualizarLEDs(int modo, int conteo){//dependiendo el modo prendemos LEDS
	if(modo == 0){// prendemos un led dependiendo el conteo
		if(conteo == 0){
			GPIOG->ODR |=(1<<1);
		}else{
			GPIOG->ODR &= (~(1<<1));
		}if(conteo == 1){
			GPIOF->ODR |=(1<<9);
		}else{
			GPIOF->ODR &= (~(1<<9));
		}if(conteo == 2){
			GPIOF->ODR |=(1<<7);
		}else{
			GPIOF->ODR &= (~(1<<7));
		}if(conteo == 3){
			GPIOF->ODR |=(1<<8);
		}else{
			GPIOF->ODR &= (~(1<<8));
		} if(conteo == 4){
			GPIOE->ODR |=(1<<3);
		}else{
			GPIOE->ODR &= (~(1<<3));
		}
	}else if(modo == 1){//prendemos leds mayores que un numero
		if(conteo >= 1){
			GPIOG->ODR |=(1<<1);
		}else{
			GPIOG->ODR &= (~(1<<1));
		}
		if(conteo>=2){
			GPIOF->ODR |=(1<<9);
		}else{
			GPIOF->ODR &= (~(1<<9));
		}
		if(conteo>=3){
			GPIOF->ODR |=(1<<7);
		}else{
			GPIOF->ODR &= (~(1<<7));
		}
		if(conteo>=4){
			GPIOF->ODR |=(1<<8);
		}else{
			GPIOF->ODR &= (~(1<<8));
		}
		if(conteo>=5){
			GPIOE->ODR |=(1<<3);
		}else{
			GPIOE->ODR &= (~(1<<3));
		}
	}else if(modo == 2){//prendemos leds de manera binaria siguiendo un agrupamiento
		if(conteo%2 !=0){
			GPIOG->ODR |=(1<<1);
		}else{
			GPIOG->ODR &= (~(1<<1));
		}
		if(conteo>1 && conteo <4 || conteo>5 && conteo < 8 || conteo>9 && conteo<12 || conteo>13 &&conteo<16 || conteo >17 && conteo< 20 || conteo>21 && conteo<24 ||conteo >25 && conteo <28 ||conteo>29 && conteo <32){
			GPIOF->ODR |=(1<<9);
		}else{
			GPIOF->ODR &= (~(1<<9));
		}
		if((conteo>3&&conteo<8)||(conteo>11&&conteo<16)||(conteo>19&&conteo<24)||(conteo>27&&conteo<32)){
			GPIOF->ODR |=(1<<7);
		}else{
			GPIOF->ODR &= (~(1<<7));
		}
		if(((conteo>7)&&(conteo<16))||((conteo>23)&&(conteo<32))){
			GPIOF->ODR |=(1<<8);
		}else{
			GPIOF->ODR &= (~(1<<8));
		}
		if(conteo>15){
			GPIOE->ODR |=(1<<3);
		}else{
			GPIOE->ODR &= (~(1<<3));
		}
	}
}

int DectectorFlancoPos(int* Up, int* Down, int* estado, int* transicion, int* time){//detectamos un pulso con la espera
	GPIOB->ODR |=(1<<14);
	if(*estado != *transicion){//chequea si se esta tratando de cambiar de estado
		*time = HAL_GetTick();//de ser asi capturamos el momento que paso
		*transicion = 1;// actualizamos un posible cambio
		GPIOB->ODR |=(1<<14);
	}
	if(*time + 75 < HAL_GetTick()  && !*estado){// si al transcurrir 50ms
		if(*Up||*Down){//y tenemos un boton presionado quiere decir que paso el debuncing
			//y si estaba presionado
			return 1;
		}
	}
	return 0;
}

void ContarPulso(int* Count, int UpPush, int DownPush, int Max){//Cuenta arriba o abajo
	//dependiendo si esta up o down y no pasa de Max
	if(UpPush == 1){
		if(*Count < Max ){
			*Count+=1;
		}
	}else if(DownPush == 1){
		if(*Count > 0){
			*Count-=1;
		}
	}
}
void ActivarIO(){//encendemos y apagamos puertos, GPIOS, LEDs y push-bot integrados
	RCC->AHB1ENR |= (1<<0);//PORTS A
	RCC->AHB1ENR |= (1<<1);//PORTS B
	RCC->AHB1ENR |= (1<<2);//PORTS C
	RCC->AHB1ENR |= (1<<4);
	RCC->AHB1ENR |= (1<<5);//PORTS F
	RCC->AHB1ENR |= (1<<6);//PORTS G
	RCC->AHB1ENR |= (1<<7);
	//C8 Pin PF14 OUTPUT LED 1SEG
	GPIOF->MODER |=(1<<28);
	GPIOF->MODER &= (~(1<<29));
	GPIOF->OTYPER &= (~(1<<14));
	GPIOF->OSPEEDR &= (3<<28);
	//LED 1 C9 PG1
	GPIOG->MODER |=(1<<2);
	GPIOG->MODER &= (~(1<<3));
	GPIOG->OTYPER &= (~(1<<1));
	GPIOG->OSPEEDR &= (3<<2);
	//LED 2 C9 PF9
	GPIOF->MODER |=(1<<18);
	GPIOF->MODER &= (~(1<<19));
	GPIOF->OTYPER &= (~(1<<9));
	GPIOF->OSPEEDR &= (3<<18);
	//LED 3 C9 PF7
	GPIOF->MODER |=(1<<14);
	GPIOF->MODER &= (~(1<<15));
	GPIOF->OTYPER &= (~(1<<7));
	GPIOF->OSPEEDR &= (3<<14);
	//LED 4 C9 PF8
	GPIOF->MODER |=(1<<16);
	GPIOF->MODER &= (~(1<<17));
	GPIOF->OTYPER &= (~(1<<8));
	GPIOF->OSPEEDR &= (3<<16);
	//LED 5 C9 PE3
	GPIOE->MODER |=(1<<6);
	GPIOE->MODER &= (~(1<<7));
	GPIOE->OTYPER &= (~(1<<3));
	GPIOE->OSPEEDR &= (3<<6);
	//PUSH INPUT UP PF12
	GPIOC->MODER &= ~(3<<20);
	GPIOC->OSPEEDR &= (3<<20);
	GPIOC->PUPDR &= ~(3<<20);
	//PUSH INPUT DW PE4
	GPIOC->MODER &= ~(3<<12);
	GPIOC->OSPEEDR &= (3<<12);
	GPIOC->PUPDR &= ~(3<<12);
	//led azul
	GPIOB->MODER |=(1<<14);
	GPIOB->MODER &= (~(1<<15));
	GPIOB->OTYPER &= (~(1<<7));
	GPIOB->OSPEEDR &= (3<<14);
	//led rojo
	GPIOB->MODER |=(1<<28);
	GPIOB->MODER &= (~(1<<29));
	GPIOB->OTYPER &= (~(1<<14));
	GPIOB->OSPEEDR &= (3<<28);
	//led verde
	GPIOB->MODER |=(1<<0);
	GPIOB->MODER &= (~(1<<1));
	GPIOB->OTYPER &= (~(1<<0));
	GPIOB->OSPEEDR &= (3<<0);
	//Push pin
	GPIOC->MODER &= ~(3<<26);
	GPIOC->OSPEEDR &= (3<<26);
	GPIOC->PUPDR &= ~(3<<26);
}
