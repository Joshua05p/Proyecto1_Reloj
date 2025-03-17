;
; PROYECTO1.asm
;
; Created: 27/02/2025 22:39:57
; Author : perez
;

.include "M328PDEF.inc" // Include definitions specific to ATMega328P
.cseg


//===================================================================================================================================
.org 0x0000
	RJMP PILA

.org 0x0006
    IN      LEC_BOTON, PINB		//Leer el estado de los botones
	SBRS    LEC_BOTON, 0		//Si PB0 está presionado, cambiar modo
    RJMP    CAMBIAR_MODOS	
	CPI		MODOS, 0x00			//VERIFICA EL MODO ACTUAL
	BREQ	MOSTRAR_HORA
	CPI		MODOS, 0x01
	BREQ	MOSTRAR_FECHA
	CPI		MODOS, 0x02
	BREQ	CAMBIAR_MINUTOS
	CPI		MODOS, 0x03
	BREQ	CAMBIAR_HORAS
	CPI		MODOS, 0x04
	BREQ	CAMBIAR_DIAS
	CPI		MODOS, 0x05
	BREQ	CAMBIAR_MESES
	CPI		MODOS, 0x06
	BREQ	CAMBIAR_MINUTOS_A
	CPI		MODOS, 0x07
	BREQ	CAMBIAR_HORAS_A
	RETI
//===================================================================================================================================

//===================================================================================================================================
.org 0x001A
	RJMP INTERRUPCION_TIMER1
//===================================================================================================================================

//===================================================================================================================================
.org 0x0020
//VERIFICA EL MODO ACTUAL
	CPI		MODOS, 0x00
	BREQ	HORA
	CPI		MODOS, 0x01
	BREQ	FECHA
	CPI		MODOS, 0x02
	BREQ	HORA1
	CPI		MODOS, 0x03
	BREQ	HORA1
	CPI		MODOS, 0x04
	BREQ	FECHA1
	CPI		MODOS, 0x05
	BREQ	FECHA1
	CPI		MODOS, 0x06
	BREQ	ALARMA
	CPI		MODOS, 0x07
	BREQ	ALARMA
	RETI
//===================================================================================================================================
//INTERRUPCIONES PORTB
//===================================================================================================================================
MOSTRAR_HORA:
	LDI		R16, (1<<CS12) | (1<<CS10)
	STS		TCCR1B, R16											//Habilita el timer1
    LDI     R16, (1 << PCINT0) | (0 << PCINT1) | (0 << PCINT2)  //Deshabilita la interrupciones del PORTB
    STS     PCMSK0, R16											//Inicia el timer1
	LDI		R16, 0x1B
    STS		TCNT1H, R16
    LDI		R16, 0x1E
    STS		TCNT1L, R16
	RETI

MOSTRAR_FECHA:
	RETI

CAMBIAR_MINUTOS:
    LDI     R16, (1 << PCINT0) | (1 << PCINT1) | (1 << PCINT2)  //Habilitar los pines 0, 1, 2
    STS     PCMSK0, R16
	LDI		R16, (0<<CS12) | (0<<CS10)
	STS		TCCR1B, R16											//Deshabilita el timer1
	RJMP	CAMBIAR_MINUTOS1
	RETI
CAMBIAR_HORAS:
	RJMP	CAMBIAR_HORAS1
	RETI
CAMBIAR_DIAS:
	RJMP	CAMBIAR_DIAS1
	RETI
CAMBIAR_MESES:
	RJMP	CAMBIAR_MESES1
	RETI
CAMBIAR_MINUTOS_A:
	RJMP	CAMBIAR_MINUTOS_ALARMA1
	RETI
CAMBIAR_HORAS_A:
	RJMP	CAMBIAR_HORAS_ALARMA1
	RETI
//===================================================================================================================================

//INTERRUPCIONES TIMER0
//===================================================================================================================================
HORA:
	SBI		PORTB, 5				//Apaga el led azul
	RJMP	MOSTRAR_HORA1
FECHA:
	SBI		PORTB, 3				//Apaga el led rojo
	RJMP	MOSTRAR_FECHA1
ALARMA:
	RJMP	MOSTRAR_ALARMA1
HORA1:
	SBI		PORTB, 5				//Apaga el led azul
	RJMP	MOSTRAR_HORA_CONFIG
FECHA1:
	SBI		PORTB, 3				//Apaga el led rojo
	RJMP	MOSTRAR_FECHA_CONFIG	
//===================================================================================================================================

PILA:
	LDI		R16, LOW(RAMEND)
	OUT		SPL, R16
	LDI		R16, HIGH(RAMEND)
	OUT		SPH, R16

	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)


//Configura los puertos
	LDI     R16, 0b11111000
    OUT     DDRB, R16
	LDI     R16, 0b00000111
    OUT     PORTB, R16
	LDI     R16, 0b11111111
    OUT     DDRD, R16
	LDI     R16, 0b11111111
    OUT     DDRC, R16

	CBI		PORTB, 4


//Interrupciones del PORTB
    LDI     R16, (1 << PCIE0)
    STS     PCICR, R16
    LDI     R16, (1 << PCINT0) | (0 << PCINT1) | (0 << PCINT2)  //Habilitar los pines 0
    STS     PCMSK0, R16

//Interrupciones del timer0
    LDI     R16, (1 << TOIE0)
    STS     TIMSK0, R16
    LDI     R16, (1 << TOV0)
    STS     TIFR0, R16

//Interrupciones del timer1
	LDI     R16, (1 << TOIE1)
	STS     TIMSK1, R16
	LDI     R16, (1 << TOV1)
	STS     TIFR1, R16

//Configuracion del prescaler
	LDI		R16, (1 << CLKPCE)
	STS		CLKPR, R16			// Habilitar cambio de PRESCALER
	LDI		R16, 0b00000100		//1MHz
	STS		CLKPR, R16

	LDI		R16, (1<<CS02) | (1<<CS00)
	OUT		TCCR0B, R16		// Setear prescaler del TIMER 0 a 1024
	LDI		R16, 251		// INTERRUPCION CADA 5 ms
	OUT		TCNT0, R16

	LDI		R16, (1<<CS12) | (1<<CS10)
	STS		TCCR1B, R16 // Setear prescaler del TIMER 1 a 1024

    LDI		R16, 0x1B	// VALOR INICIAL DEL TIMER 1
    STS		TCNT1H, R16
    LDI		R16, 0x1E
    STS		TCNT1L, R16 // INTERRUPCION CADA 60 s


//VALORES INICIALES: VARIABLES 
	LDI		R16, 0x00

	MOV		CONTADOR_MINUTOS1, R16
	MOV		CONTADOR_HORAS1, R16
	MOV		ACTUALIZAR_U_HORAS, R16
	MOV		CONTADOR_MINUTOS, R16
	MOV		CONTADOR_MINUTOS1, R16
	MOV		CONTADOR_HORAS1, R16
	MOV		ACTUALIZAR_U_HORAS, R16
	MOV		LEC_BOTON, R16
	MOV		LED_CENTRAL, R16
	MOV		CONTADOR_DIAS2, R16
	MOV		CONTADOR_MINUTOS, R16
	MOV		CONTADOR_MESES1, R16
	MOV		CONTADOR_ALARMA1, R16
	MOV		CONTADOR_ALARMA2, R16
	MOV		CONTADOR_ALARMA3, R16
	MOV		CONTADOR_ALARMA4, R16
	OUT		PORTC, R16

//VALORES INICIALES: DISPLAYS

	LPM		UNIDADES_MINUTOS, Z
	LPM		DECENAS_MINUTOS, Z
	LPM		UNIDADES_HORAS, Z
	LPM		DECENAS_HORAS, Z
	ADIW	ZL, 1
	LPM		UNIDADES_DIAS, Z
	LPM		UNIDADES_MESES, Z
	SBIW	ZL, 1
	LPM		DECENAS_DIAS, Z
	LPM		DECENAS_MESES, Z
	LPM		UNIDADES_ALARMA_M, Z
	LPM		DECENAS_ALARMA_M, Z
	LPM		UNIDADES_ALARMA_H, Z
	LPM		DECENAS_ALARMA_H, Z

	LDI R16, 0x01
	MOV CONTADOR_DIAS1, R16
	MOV CONTADOR_MESES, R16
	MOV PUNTERO_DISPLAYS, R16

	LDI MODOS, 0x00
	SEI

MAIN:
	IN		LEC_BOTON, PINB
	SBRC	LEC_BOTON, 4
	RJMP	APAGAR_ALARMA
	RJMP	MAIN

APAGAR_ALARMA:
	SBRS	LEC_BOTON, 1
	CBI		PORTB, 4
	SBRS	LEC_BOTON, 2
	CBI		PORTB, 4
	RJMP	MAIN


//CAMBIA DE MODOS
CAMBIAR_MODOS:
	INC MODOS
	CPI MODOS, 0x08		//Limita los modos a 8
	BREQ REINICIO_MODO
	RETI
REINICIO_MODO:
	LDI MODOS, 0x00		//Reinicia los modos
	RETI

//MOSTRAR HORA
//===================================================================================================================================
//===================================================================================================================================
INTERRUPCION_TIMER1:
    SBI		TIFR1, TOV1			// Limpiar bandera de overflow
    LDI		R16, 0x1B			//VALOR INICIAL DEL TIMER 1
    STS		TCNT1H, R16
    LDI		R16, 0x1E
    STS		TCNT1L, R16
	LDI		ZH, HIGH(LISTA<<1)	//Mueve el puntero a la posicion 0
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_MINUTOS	//Mueve el puntero a la posicion de la unidades de minutos	
	LPM		UNIDADES_MINUTOS, Z		
	CPI		UNIDADES_MINUTOS, 0x90	//Compara si unidades minutos es 9
	BREQ	REINICIO_TIMER1			//Salta si unidades minutos es 9
	ADIW	Z, 1					//Suma 1 a la direccion de Z
	LPM		UNIDADES_MINUTOS, Z		
	INC		CONTADOR_MINUTOS
	CP		DECENAS_HORAS, DECENAS_ALARMA_H
	BRNE	SKIP_ALARMA
	CP		UNIDADES_HORAS, UNIDADES_ALARMA_H
	BRNE	SKIP_ALARMA
	CP		DECENAS_MINUTOS, DECENAS_ALARMA_M
	BRNE	SKIP_ALARMA
	CP		UNIDADES_MINUTOS, UNIDADES_ALARMA_M
	BRNE	SKIP_ALARMA
	SBI		PINB, 4
SKIP_ALARMA:
	RETI

REINICIO_TIMER1:
	LDI		R16, 0x00				//Reinicia la unidades de minutos
	MOV		CONTADOR_MINUTOS, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		UNIDADES_MINUTOS, Z
	CPI		DECENAS_MINUTOS, 0x92		//Compara si las decenas minutos es 5
	BREQ	REINICIO_DECENAS			//Salta si decenas minutos es 5
	INC		CONTADOR_MINUTOS1			
	ADD		ZL, CONTADOR_MINUTOS1		//Incrementa en 1 a las decenas de minutos
	LPM		DECENAS_MINUTOS, Z
	SUB		ZL, CONTADOR_MINUTOS1
	CP		DECENAS_HORAS, DECENAS_ALARMA_H
	BRNE	SKIP_ALARMA1
	CP		UNIDADES_HORAS, UNIDADES_ALARMA_H
	BRNE	SKIP_ALARMA1
	CP		DECENAS_MINUTOS, DECENAS_ALARMA_M
	BRNE	SKIP_ALARMA1
	CP		UNIDADES_MINUTOS, UNIDADES_ALARMA_M
	BRNE	SKIP_ALARMA1
	SBI		PINB, 4
SKIP_ALARMA1:
	RETI

REINICIO_DECENAS:
	LDI		R16, 0x00					//Reinicia los minutos a 00
	MOV		CONTADOR_MINUTOS1, R16
	LPM		DECENAS_MINUTOS, Z

	CPI		DECENAS_HORAS, 0xA4			//Verifica si la decenas horas es 2
	BREQ	VERIFICAR					//Salta si decenas horas es 2
	RJMP	CONTINUAR					//Salta si no es 2

VERIFICAR:
	CPI		UNIDADES_HORAS, 0xB0		//Verifica si decenas horas es 3
	BREQ	REINICIO_DECENAS_HORAS		//Salta si es 3

CONTINUAR:
	CPI		UNIDADES_HORAS, 0x90		//Verifica si unidades horas es 9
	BREQ	REINICIO_UNIDADES_HORAS		//Salta si es 9
	INC		ACTUALIZAR_U_HORAS
	ADD		ZL, ACTUALIZAR_U_HORAS
	LPM		UNIDADES_HORAS, Z			//Suma 1 a las unidades de horas
	SUB		ZL, ACTUALIZAR_U_HORAS
	RETI

REINICIO_UNIDADES_HORAS:
	LDI		R16, 0x00					//Reinicia las unidades de horas
	MOV		ACTUALIZAR_U_HORAS, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		UNIDADES_HORAS, Z
	CPI		DECENAS_HORAS, 0xA4			//Verifica si decenas horas es 2
	BREQ	REINICIO_DECENAS_HORAS
	INC		CONTADOR_HORAS1				//Suma 1 a las decenas de horas
	ADD		ZL, CONTADOR_HORAS1
	LPM		DECENAS_HORAS, Z
	SUB		ZL, CONTADOR_HORAS1
	CP		DECENAS_HORAS, DECENAS_ALARMA_H
	BRNE	SKIP_ALARMA2
	CP		UNIDADES_HORAS, UNIDADES_ALARMA_H
	BRNE	SKIP_ALARMA2
	CP		DECENAS_MINUTOS, DECENAS_ALARMA_M
	BRNE	SKIP_ALARMA2
	CP		UNIDADES_MINUTOS, UNIDADES_ALARMA_M
	BRNE	SKIP_ALARMA2
	SBI		PINB, 4
SKIP_ALARMA2:
	RETI

REINICIO_DECENAS_HORAS:
	LDI		R16, 0x00
	MOV		ACTUALIZAR_U_HORAS, R16		//Reinicia la hora a 00:00
	MOV		CONTADOR_HORAS1, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		DECENAS_HORAS, Z
	LPM		UNIDADES_HORAS, Z
	
	CPI		DECENAS_DIAS, 0xB0			//Verifica si decenas dias es igual a 3
	BREQ	VERIFICAR_DIAS				
	CPI		UNIDADES_DIAS, 0x90			//Verifica si unidades dias es 9
	BREQ	REINICIO_DIAS

	INC		CONTADOR_DIAS1				//Suma 1 a las unidades dias si no es 9
	ADD		ZL, CONTADOR_DIAS1
	LPM		UNIDADES_DIAS, Z
	SUB		ZL, CONTADOR_DIAS1

	CP		DECENAS_HORAS, DECENAS_ALARMA_H
	BRNE	SKIP_ALARMA3
	CP		UNIDADES_HORAS, UNIDADES_ALARMA_H
	BRNE	SKIP_ALARMA3
	CP		DECENAS_MINUTOS, DECENAS_ALARMA_M
	BRNE	SKIP_ALARMA3
	CP		UNIDADES_MINUTOS, UNIDADES_ALARMA_M
	BRNE	SKIP_ALARMA3
	SBI		PINB, 4
SKIP_ALARMA3:
	RETI

REINICIO_DIAS:
	LDI		R16, 0x00					//Reinicia las unidades dias 
	MOV		CONTADOR_DIAS1, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		UNIDADES_DIAS, Z
	INC		CONTADOR_DIAS2
	ADD		ZL, CONTADOR_DIAS2
	LPM		DECENAS_DIAS, Z				//Suma 1 a las decenas dias
	SUB		ZL, CONTADOR_DIAS2
	RETI

VERIFICAR_DIAS:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		DECENAS_DIAS, Z
	LDI		R16, 0x01
	MOV		CONTADOR_DIAS1, R16
	LDI		R16, 0x00
	MOV		CONTADOR_DIAS2, R16
	ADD		ZL, CONTADOR_DIAS1
	LPM		UNIDADES_DIAS, Z			//Reinicia los dias a 01:xx
	SUB		ZL, CONTADOR_DIAS1
	CPI		DECENAS_MESES, 0xF9			//Verfica si decenas meses es 1
	BREQ	VERIFICAR_MESES
	CPI		UNIDADES_MESES, 0x90		//Verifica si uniades meses es 9
	BREQ	REINICIO_U_MESES
	INC		CONTADOR_MESES
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z			//Suma 1 si unidades meses no es 9
	SUB		ZL, CONTADOR_MESES
	RETI

REINICIO_U_MESES:
	LDI		R16, 0x00
	MOV		CONTADOR_MESES, R16		
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)

	LPM		UNIDADES_MESES, Z			//Reinicia la unidades de meses 
	INC		CONTADOR_MESES1
	ADD		ZL, CONTADOR_MESES1
	LPM		DECENAS_MESES, Z			//Suma 1 a las decenas de meses
	SUB		ZL, CONTADOR_MESES1
	RETI
VERIFICAR_MESES:
	CPI		UNIDADES_MESES, 0xA4		//Verifica las unidades de meses si decenas de meses es 2
	BREQ	REINICIO_TOTAL_MESES
	INC		CONTADOR_MESES				//Suma 1 si el mes no es 12
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z
	SUB		ZL, CONTADOR_MESES
	RETI

REINICIO_TOTAL_MESES:
	LDI		R16, 0x01					//Reinicia los meses 0 xx:01
	MOV		CONTADOR_MESES, R16
	LDI		R16, 0x00
	MOV		CONTADOR_MESES1, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		DECENAS_MESES, Z
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z
	SUB		ZL, CONTADOR_MESES
	RETI

	
//===================================================================================================================================
//===================================================================================================================================

//CAMBIAR MINUTOS
//===================================================================================================================================
//===================================================================================================================================
CAMBIAR_MINUTOS1:
	SBRS    LEC_BOTON, 1       //Si PB0 está presionado, cambiar modo
    RJMP    INCREMENTO_MINUTOS
	SBRS    LEC_BOTON, 2       //Si PB0 está presionado, cambiar modo
    RJMP    DECREMENTO_MINUTOS
	RETI
INCREMENTO_MINUTOS:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_MINUTOS
	LPM		UNIDADES_MINUTOS, Z
	CPI		UNIDADES_MINUTOS, 0x90
	BREQ	REINICIO_UNIDADES_MINUTOS1
	ADIW	Z, 1
	LPM		UNIDADES_MINUTOS, Z
	INC		CONTADOR_MINUTOS
	RETI

REINICIO_UNIDADES_MINUTOS1:
	LDI		R16, 0x00
	MOV		CONTADOR_MINUTOS, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		UNIDADES_MINUTOS, Z
	CPI		DECENAS_MINUTOS, 0x92
	BREQ	REINICIO_DECENAS_MINUTOS1
	INC		CONTADOR_MINUTOS1
	ADD		ZL, CONTADOR_MINUTOS1
	LPM		DECENAS_MINUTOS, Z
	SUB		ZL, CONTADOR_MINUTOS1
	RETI
REINICIO_DECENAS_MINUTOS1:
	LDI		R16, 0x00
	MOV		CONTADOR_MINUTOS1, R16
	LPM		DECENAS_MINUTOS, Z
	RETI

DECREMENTO_MINUTOS:
	CPI		UNIDADES_MINUTOS, 0xC0
	BREQ	REINICIO_UNIDADES_MINUTOS
	SBIW	Z, 1
	LPM		UNIDADES_MINUTOS, Z
	DEC		CONTADOR_MINUTOS
	RETI

REINICIO_UNIDADES_MINUTOS:
	LDI		R16, 0x09
	MOV		CONTADOR_MINUTOS, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADIW	Z, 9
	LPM		UNIDADES_MINUTOS, Z
	CPI		DECENAS_MINUTOS, 0xC0
	BREQ	REINICIO_DECENAS_MINUTOS
	SBIW	Z, 9
	DEC		CONTADOR_MINUTOS1
	ADD		ZL, CONTADOR_MINUTOS1
	LPM		DECENAS_MINUTOS, Z
	SUB		ZL, CONTADOR_MINUTOS1
	ADIW	Z, 9
	RETI
REINICIO_DECENAS_MINUTOS:
	LDI		R16, 0x05
	MOV		CONTADOR_MINUTOS1, R16
	SBIW	Z, 4
	LPM		DECENAS_MINUTOS, Z
	ADIW	Z, 4
	RETI
//===================================================================================================================================
//===================================================================================================================================


//CAMBIAR HORAS
//===================================================================================================================================
//===================================================================================================================================
CAMBIAR_HORAS1:
	SBRS    LEC_BOTON, 1       //Si PB0 está presionado, cambiar modo
    RJMP    INCREMENTO_HORAS
	SBRS    LEC_BOTON, 2       //Si PB0 está presionado, cambiar modo
    RJMP    DECREMENTO_HORAS
	RETI
INCREMENTO_HORAS:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, ACTUALIZAR_U_HORAS
	LPM		UNIDADES_HORAS, Z
	CPI		DECENAS_HORAS, 0xA4
	BREQ	VERIFICAR2
	CPI		UNIDADES_HORAS, 0x90
	BREQ	REINICIO_U_HORAS1
	ADIW	Z, 1
	LPM		UNIDADES_HORAS, Z
	INC		ACTUALIZAR_U_HORAS
	RETI
VERIFICAR2:
	CPI		UNIDADES_HORAS, 0xB0
	BREQ	CAMBIAR_PUNTERO1
	ADIW	Z, 1
	LPM		UNIDADES_HORAS, Z
	INC		ACTUALIZAR_U_HORAS
	RETI

REINICIO_U_HORAS1:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		UNIDADES_HORAS, Z
	LDI		R16, 0x00
	MOV		ACTUALIZAR_U_HORAS, R16
	CPI		DECENAS_HORAS, 0xA4
	BREQ	REINICIO_DECENAS_HORAS1
	INC		CONTADOR_HORAS1
	ADD		ZL, CONTADOR_HORAS1
	LPM		DECENAS_HORAS, Z
	SUB		ZL, CONTADOR_HORAS1
	RETI

CAMBIAR_PUNTERO1:
	SBIW	Z, 3
	LPM		UNIDADES_HORAS, Z
	LDI		R16, 0x00
	MOV		ACTUALIZAR_U_HORAS, R16

REINICIO_DECENAS_HORAS1:
	LDI		R16, 0x00
	MOV		CONTADOR_HORAS1, R16
	LPM		DECENAS_HORAS, Z
	RETI

DECREMENTO_HORAS:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, ACTUALIZAR_U_HORAS
	LPM		UNIDADES_HORAS, Z
	CPI		UNIDADES_HORAS, 0xC0
	BREQ	REINICIO_UNIDADES_HORAS2
	DEC		ACTUALIZAR_U_HORAS
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, ACTUALIZAR_U_HORAS
	LPM		UNIDADES_HORAS, Z
	RETI

REINICIO_UNIDADES_HORAS2:
	CPI		DECENAS_HORAS, 0xC0
	BREQ	VERIFICAR3
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADIW	Z, 9
	LPM		UNIDADES_HORAS, Z
	SBIW	Z, 9
	LDI		R16, 0x09
	MOV		ACTUALIZAR_U_HORAS, R16
	DEC		CONTADOR_HORAS1
	ADD		ZL, CONTADOR_HORAS1
	LPM		DECENAS_HORAS, Z
	SUB		ZL, CONTADOR_HORAS1
	ADIW	Z, 9
	RETI

VERIFICAR3:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADIW	Z, 2
	LPM		DECENAS_HORAS, Z
	ADIW	Z, 1
	LPM		UNIDADES_HORAS, Z
	LDI		R16, 0x03
	MOV		ACTUALIZAR_U_HORAS, R16
	LDI		R16, 0x02
	MOV		CONTADOR_HORAS1, R16
	RETI
REINICIO_DECENAS_HORAS2:
	LDI		R16, 0x03
	MOV		ACTUALIZAR_U_HORAS, R16
	LDI		R16, 0x02
	MOV		CONTADOR_HORAS1, R16
	SBIW	Z, 1
	LPM		DECENAS_HORAS, Z
	ADIW	Z, 1
	RETI
//===================================================================================================================================
//===================================================================================================================================


//CAMBIAR DIAS
//===================================================================================================================================
//===================================================================================================================================
CAMBIAR_DIAS1:
	SBRS    LEC_BOTON, 1       //Si PB0 está presionado, cambiar modo
    RJMP    INCREMENTO_DIAS
	SBRS    LEC_BOTON, 2       //Si PB0 está presionado, cambiar modo
    RJMP    DECREMENTO_DIAS
	RETI
INCREMENTO_DIAS:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_DIAS1
	LPM		UNIDADES_DIAS, Z
	CPI		DECENAS_DIAS, 0xB0
	BREQ	REINICIO_DIAS_TOTAL_BOTONES
	CPI		UNIDADES_DIAS, 0x90
	BREQ	REINICIO_UNIDADES_DIAS_BOTONES
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	INC		CONTADOR_DIAS1
	ADD		ZL, CONTADOR_DIAS1
	LPM		UNIDADES_DIAS, Z
	SUB		ZL, CONTADOR_DIAS1
	RETI

REINICIO_UNIDADES_DIAS_BOTONES:
	LDI		R16, 0x00
	MOV		CONTADOR_DIAS1, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		UNIDADES_DIAS, Z
	INC		CONTADOR_DIAS2
	ADD		ZL, CONTADOR_DIAS2
	LPM		DECENAS_DIAS, Z
	SUB		ZL, CONTADOR_DIAS2
	RETI

REINICIO_DIAS_TOTAL_BOTONES:
	LDI		R16, 0x01
	MOV		CONTADOR_DIAS1, R16
	MOV		CONTADOR_DIAS2, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		DECENAS_DIAS, Z
	ADD		ZL, CONTADOR_DIAS1
	LPM		UNIDADES_DIAS, Z
	SUB		ZL, CONTADOR_DIAS1
	RETI


DECREMENTO_DIAS:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_DIAS1
	LPM		UNIDADES_DIAS, Z
	CPI		DECENAS_DIAS, 0xC0
	BREQ	VERIFICAR_DIAS_BOTONES
	CPI		UNIDADES_DIAS, 0xC0
	BREQ	REINICIO_DIAS_BOTONES
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	DEC		CONTADOR_DIAS1
	ADD		ZL, CONTADOR_DIAS1
	LPM		UNIDADES_DIAS, Z
	SUB		ZL, CONTADOR_DIAS1
	RETI
REINICIO_DIAS_BOTONES:
	LDI		R16, 0x09
	MOV		CONTADOR_DIAS1, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_DIAS1
	LPM		UNIDADES_DIAS, Z
	SUB		ZL, CONTADOR_DIAS1

	DEC		CONTADOR_DIAS2
	ADD		ZL, CONTADOR_DIAS2
	LPM		DECENAS_DIAS, Z
	SUB		ZL, CONTADOR_DIAS2
	RETI
VERIFICAR_DIAS_BOTONES:
	CPI		UNIDADES_DIAS, 0xF9
	BREQ	REINICIO_TOTAL_DIAS_BOTONES1

	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		UNIDADES_DIAS, Z
	DEC		CONTADOR_DIAS1
	ADD		ZL, CONTADOR_DIAS1
	LPM		UNIDADES_DIAS, Z
	SUB		ZL, CONTADOR_DIAS1
	RETI

REINICIO_TOTAL_DIAS_BOTONES1:
	LDI		R16, 0x03
	MOV		CONTADOR_DIAS2, R16
	LDI		R16, 0x00
	MOV		CONTADOR_DIAS1, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		UNIDADES_DIAS, Z
	ADD		ZL, CONTADOR_DIAS2
	LPM		DECENAS_DIAS, Z
	SUB		ZL, CONTADOR_DIAS2
	RETI

	
REINICIO_DECENAS_TOTAL_BOTONES:
	LDI		R16, 0x03
	MOV		CONTADOR_DIAS2, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_DIAS2
	LPM		DECENAS_DIAS, Z
	SUB		ZL, CONTADOR_DIAS2
	RETI



//===================================================================================================================================
//===================================================================================================================================



//CAMBIAR MESES
//===================================================================================================================================
//===================================================================================================================================
CAMBIAR_MESES1:
	SBRS    LEC_BOTON, 1       //Si PB0 está presionado, cambiar modo
    RJMP    INCREMENTO_MESES
	SBRS    LEC_BOTON, 2       //Si PB0 está presionado, cambiar modo
    RJMP    DECREMENTO_MESES
	RETI
INCREMENTO_MESES:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z
	CPI		DECENAS_MESES, 0xF9
	BREQ	VERIFICAR_MESES_INC
	CPI		UNIDADES_MESES, 0x90
	BREQ	REINICIO_U_MESES_INC
	INC		CONTADOR_MESES
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z
	RETI

VERIFICAR_MESES_INC:
	CPI		UNIDADES_MESES, 0xA4
	BREQ	REINICIO_TOTAL_MESES_INC
	INC		CONTADOR_MESES
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z
	SUB		ZL, CONTADOR_MESES
	RETI

REINICIO_U_MESES_INC:
	LDI		R16, 0x00
	MOV		CONTADOR_MESES, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		UNIDADES_MESES, Z
	INC		CONTADOR_MESES1
	ADD		ZL, CONTADOR_MESES1
	LPM		DECENAS_MESES, Z
	SUB		ZL, CONTADOR_MESES1
	RETI

REINICIO_TOTAL_MESES_INC:
	LDI		R16, 0x01
	MOV		CONTADOR_MESES, R16
	LDI		R16, 0x00
	MOV		CONTADOR_MESES1, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		DECENAS_MESES, Z
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z
	SUB		ZL, CONTADOR_MESES
	RETI

DECREMENTO_MESES:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z
	SUB		ZL, CONTADOR_MESES

	CPI		DECENAS_MESES, 0xC0
	BREQ	REINICIO_MESES_DEC1
	CPI		DECENAS_MESES, 0xF9
	BREQ	REINICIO_MESES_DEC2

	CPI		UNIDADES_MESES, 0xC0
	BREQ	REINICIAR_U_MESES_DEC
	DEC		CONTADOR_MESES
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z
	SUB		ZL, CONTADOR_MESES
	RETI

REINICIO_MESES_DEC1:
	CPI		UNIDADES_MESES, 0xF9
	BREQ	REINICIAR_U_MESES_DEC
	DEC		CONTADOR_MESES
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z
	SUB		ZL, CONTADOR_MESES
	RETI
REINICIO_MESES_DEC2:
	CPI		UNIDADES_MESES, 0xC0
	BREQ	REINICIAR_U_MESES_DEC1
	DEC		CONTADOR_MESES
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z
	SUB		ZL, CONTADOR_MESES


	RETI

REINICIAR_U_MESES_DEC1:
	LDI		R16, 0x09
	MOV		CONTADOR_MESES, R16
	LDI		R16, 0x00
	MOV		CONTADOR_MESES1, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		DECENAS_MESES, Z
	ADIW	Z, 9
	LPM		UNIDADES_MESES, Z
	SBIW	Z, 9
	RETI

REINICIAR_U_MESES_DEC:
	LDI		R16, 0x01
	MOV		CONTADOR_MESES1, R16
	LDI		R16, 0x02
	MOV		CONTADOR_MESES, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_MESES
	LPM		UNIDADES_MESES, Z
	SUB		ZL, CONTADOR_MESES
	ADIW	Z, 1
	LPM		DECENAS_MESES, Z
	SBIW	Z, 1
	RETI

//===================================================================================================================================
//===================================================================================================================================

//CONFIGURAR ALARMA
//===================================================================================================================================
//===================================================================================================================================
CAMBIAR_MINUTOS_ALARMA1:
	SBRS    LEC_BOTON, 1       //Si PB0 está presionado, cambiar modo
    RJMP    INCREMENTO_MINUTOS_ALARMA
	SBRS    LEC_BOTON, 2       //Si PB0 está presionado, cambiar modo
    RJMP    DECREMENTO_MINUTOS_ALARMA
	RETI
INCREMENTO_MINUTOS_ALARMA:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_ALARMA1
	LPM		UNIDADES_ALARMA_M, Z
	CPI		UNIDADES_ALARMA_M, 0x90
	BREQ	REINICIO_UNIDADES_MINUTOS_A
	ADIW	Z, 1
	LPM		UNIDADES_ALARMA_M, Z
	INC		CONTADOR_ALARMA1
	RETI

REINICIO_UNIDADES_MINUTOS_A:
	LDI		R16, 0x00
	MOV		CONTADOR_ALARMA1, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		UNIDADES_ALARMA_M, Z
	CPI		DECENAS_ALARMA_M, 0x92
	BREQ	REINICIO_DECENAS_MINUTOS_A
	INC		CONTADOR_ALARMA2
	ADD		ZL, CONTADOR_ALARMA2
	LPM		DECENAS_ALARMA_M, Z
	SUB		ZL, CONTADOR_ALARMA2
	RETI
REINICIO_DECENAS_MINUTOS_A:
	LDI		R16, 0x00
	MOV		CONTADOR_ALARMA2, R16
	LPM		DECENAS_ALARMA_M, Z
	RETI

DECREMENTO_MINUTOS_ALARMA:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_ALARMA1
	LPM		UNIDADES_ALARMA_M, Z
	CPI		UNIDADES_ALARMA_M, 0xC0
	BREQ	REINICIO_UNIDADES_MINUTOS1_A
	SBIW	Z, 1
	LPM		UNIDADES_ALARMA_M, Z
	DEC		CONTADOR_ALARMA1
	RETI

REINICIO_UNIDADES_MINUTOS1_A:
	LDI		R16, 0x09
	MOV		CONTADOR_ALARMA1, R16
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADIW	Z, 9
	LPM		UNIDADES_ALARMA_M, z
	CPI		DECENAS_ALARMA_M, 0xC0
	BREQ	REINICIO_DECENAS_MINUTOS1_A
	SBIW	Z, 9
	DEC		CONTADOR_ALARMA2
	ADD		ZL, CONTADOR_ALARMA2
	LPM		DECENAS_ALARMA_M, Z
	SUB		ZL, CONTADOR_ALARMA2
	ADIW	Z, 9
	RETI
REINICIO_DECENAS_MINUTOS1_A:
	LDI		R16, 0x05
	MOV		CONTADOR_ALARMA2, R16
	SBIW	Z, 4
	LPM		DECENAS_ALARMA_M, Z
	ADIW	Z, 4
	RETI
//===================================================================================================================================
//===================================================================================================================================


//CAMBIAR HORAS ALARMA
//===================================================================================================================================
//===================================================================================================================================
CAMBIAR_HORAS_ALARMA1:
	SBRS    LEC_BOTON, 1       //Si PB0 está presionado, cambiar modo
    RJMP    INCREMENTO_HORAS_ALARMA
	SBRS    LEC_BOTON, 2       //Si PB0 está presionado, cambiar modo
    RJMP    DECREMENTO_HORAS_ALARMA
	RETI
INCREMENTO_HORAS_ALARMA:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_ALARMA3
	LPM		UNIDADES_ALARMA_H, Z
	CPI		DECENAS_ALARMA_H, 0xA4
	BREQ	VERIFICAR2_A
	CPI		UNIDADES_ALARMA_H, 0x90
	BREQ	REINICIO_U_HORAS1_A
	ADIW	Z, 1
	LPM		UNIDADES_ALARMA_H, Z
	INC		CONTADOR_ALARMA3
	RETI
VERIFICAR2_A:
	CPI		UNIDADES_ALARMA_H, 0xB0
	BREQ	CAMBIAR_PUNTERO1_A
	ADIW	Z, 1
	LPM		UNIDADES_ALARMA_H, Z
	INC		CONTADOR_ALARMA3
	RETI

REINICIO_U_HORAS1_A:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	LPM		UNIDADES_ALARMA_H, Z
	LDI		R16, 0x00
	MOV		CONTADOR_ALARMA3, R16
	CPI		DECENAS_ALARMA_H, 0xA4
	BREQ	REINICIO_DECENAS_HORAS1_A
	INC		CONTADOR_ALARMA4
	ADD		ZL, CONTADOR_ALARMA4
	LPM		DECENAS_ALARMA_H, Z
	SUB		ZL, CONTADOR_ALARMA4
	RETI

CAMBIAR_PUNTERO1_A:
	SBIW	Z, 3
	LPM		UNIDADES_ALARMA_H, Z
	LDI		R16, 0x00
	MOV		CONTADOR_ALARMA3, R16

REINICIO_DECENAS_HORAS1_A:
	LDI		R16, 0x00
	MOV		CONTADOR_ALARMA4, R16
	LPM		DECENAS_ALARMA_H, Z
	RETI

DECREMENTO_HORAS_ALARMA:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_ALARMA3
	LPM		UNIDADES_ALARMA_H, Z
	CPI		UNIDADES_ALARMA_H, 0xC0
	BREQ	REINICIO_UNIDADES_HORAS2_A
	DEC		CONTADOR_ALARMA3
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADD		ZL, CONTADOR_ALARMA3
	LPM		UNIDADES_ALARMA_H, Z
	RETI

REINICIO_UNIDADES_HORAS2_A:
	CPI		DECENAS_ALARMA_H, 0xC0
	BREQ	VERIFICAR3_A
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADIW	Z, 9
	LPM		UNIDADES_ALARMA_H, Z
	SBIW	Z, 9
	LDI		R16, 0x09
	MOV		CONTADOR_ALARMA3, R16
	DEC		CONTADOR_ALARMA4
	ADD		ZL, CONTADOR_ALARMA4
	LPM		DECENAS_ALARMA_H, Z
	SUB		ZL, CONTADOR_ALARMA4
	ADIW	Z, 9
	RETI

VERIFICAR3_A:
	LDI		ZH, HIGH(LISTA<<1)
	LDI		ZL, LOW(LISTA<<1)
	ADIW	Z, 2
	LPM		DECENAS_ALARMA_H, Z
	ADIW	Z, 1
	LPM		UNIDADES_ALARMA_H, Z
	LDI		R16, 0x03
	MOV		CONTADOR_ALARMA3, R16
	LDI		R16, 0x02
	MOV		CONTADOR_ALARMA4, R16
	RETI
REINICIO_DECENAS_HORAS2_A:
	LDI		R16, 0x03
	MOV		CONTADOR_ALARMA3, R16
	LDI		R16, 0x02
	MOV		CONTADOR_ALARMA4, R16
	SBIW	Z, 1
	LPM		DECENAS_ALARMA_H, Z
	ADIW	Z, 1
	RETI


//MOSTRAR HORAS
//===================================================================================================================================
//===================================================================================================================================

MOSTRAR_HORA1:
    SBI		TIFR0, TOV0 // Limpiar bandera de overflow
	LDI		R16, 251 // INTERRUPCION CADA 5 ms
	OUT		TCNT0, R16
	MOV		R16, LED_CENTRAL
	CPI		R16, 100
	BRNE	ALTERNAR_1
	LDI		R16, 0x00
	MOV		LED_CENTRAL, R16
	SBI		PINB, 3
ALTERNAR_1:
	INC		LED_CENTRAL
    LSL		PUNTERO_DISPLAYS         ; Desplazar a la izquierda (0001 ? 0010 ? 0100 ? 1000)
	MOV		R16, PUNTERO_DISPLAYS
    CPI		R16, 0b00010000  ; Si el valor es mayor a 1000, reiniciar
    BRNE	CONTINUAR1_1
    LDI		R16, 0b00000001  ; Reiniciar a 0001
	MOV		PUNTERO_DISPLAYS, R16
CONTINUAR1_1:
    OUT		PORTC, PUNTERO_DISPLAYS  ; Enviar el valor a PORTC
	MOV		R16, PUNTERO_DISPLAYS
	CPI		R16, 0b00000001
	BREQ	DISPLAY4_1
	CPI		R16, 0b00000010
	BREQ	DISPLAY3_1
	CPI		R16, 0b00000100
	BREQ	DISPLAY2_1
	CPI		R16, 0b00001000
	BREQ	DISPLAY1_1
DISPLAY1_1:
	OUT		PORTD, UNIDADES_MINUTOS
	RETI
DISPLAY2_1:
	OUT		PORTD, DECENAS_MINUTOS
	RETI
DISPLAY3_1:
	OUT		PORTD, UNIDADES_HORAS
	RETI
DISPLAY4_1:
	OUT		PORTD, DECENAS_HORAS
	RETI
//===================================================================================================================================
//===================================================================================================================================


//MOSTRAR FECHA
//===================================================================================================================================
//===================================================================================================================================
MOSTRAR_FECHA1:
    SBI		TIFR0, TOV0 // Limpiar bandera de overflow
	LDI		R16, 251 // INTERRUPCION CADA 5 ms
	OUT		TCNT0, R16
	MOV		R16, LED_CENTRAL
	CPI		R16, 100
	BRNE	ALTERNAR
	LDI		R16, 0x00
	MOV		LED_CENTRAL, R16
	SBI		PINB, 5
ALTERNAR:
	INC		LED_CENTRAL
    LSL		PUNTERO_DISPLAYS         //Desplazar a la izquierda (0001 ? 0010 ? 0100 ? 1000)
	MOV		R16, PUNTERO_DISPLAYS
    CPI		R16, 0b00010000			//Si el valor es mayor a 1000, reiniciar
    BRNE	CONTINUAR1
    LDI		R16, 0b00000001			//Reiniciar a 0001
	MOV		PUNTERO_DISPLAYS, R16
CONTINUAR1:
    OUT		PORTC, PUNTERO_DISPLAYS		//Enviar el valor a PORTC
	MOV		R16, PUNTERO_DISPLAYS
	CPI		R16, 0b00000001
	BREQ	DISPLAY4
	CPI		R16, 0b00000010
	BREQ	DISPLAY3
	CPI		R16, 0b00000100
	BREQ	DISPLAY2
	CPI		R16, 0b00001000
	BREQ	DISPLAY1
DISPLAY1:
	OUT		PORTD, UNIDADES_MESES
	RETI
DISPLAY2:
	OUT		PORTD, DECENAS_MESES
	RETI
DISPLAY3:
	OUT		PORTD, UNIDADES_DIAS
	RETI
DISPLAY4:
	OUT		PORTD, DECENAS_DIAS
	RETI
//===================================================================================================================================

//MOSTRAR ALARMA
//===================================================================================================================================
//===================================================================================================================================
MOSTRAR_ALARMA1:
    SBI		TIFR0, TOV0 // Limpiar bandera de overflow
	LDI		R16, 251 // INTERRUPCION CADA 5 ms
	OUT		TCNT0, R16
	MOV		R16, LED_CENTRAL
	CPI		R16, 30
	BRNE	ALTERNAR_2
	LDI		R16, 0x00
	MOV		LED_CENTRAL, R16
	SBI		PINB, 3
	SBI		PINB, 5
ALTERNAR_2:
	INC		LED_CENTRAL
    LSL		PUNTERO_DISPLAYS         //Desplazar a la izquierda (0001 ? 0010 ? 0100 ? 1000)
	MOV		R16, PUNTERO_DISPLAYS
    CPI		R16, 0b00010000			//Si el valor es mayor a 1000, reiniciar
    BRNE	CONTINUAR1_2
    LDI		R16, 0b00000001			//Reiniciar a 0001
	MOV		PUNTERO_DISPLAYS, R16
CONTINUAR1_2:
    OUT		PORTC, PUNTERO_DISPLAYS  //Enviar el valor a PORTC
	MOV		R16, PUNTERO_DISPLAYS
	CPI		R16, 0b00000001
	BREQ	DISPLAY4_2
	CPI		R16, 0b00000010
	BREQ	DISPLAY3_2
	CPI		R16, 0b00000100
	BREQ	DISPLAY2_2
	CPI		R16, 0b00001000
	BREQ	DISPLAY1_2
DISPLAY1_2:
	OUT		PORTD, UNIDADES_ALARMA_M
	RETI
DISPLAY2_2:
	OUT		PORTD, DECENAS_ALARMA_M
	RETI
DISPLAY3_2:
	OUT		PORTD, UNIDADES_ALARMA_H
	RETI
DISPLAY4_2:
	OUT		PORTD, DECENAS_ALARMA_H
	RETI
//===================================================================================================================================
//MOSTRAR HORA CONFIGURACION
//===================================================================================================================================
MOSTRAR_HORA_CONFIG:
    SBI		TIFR0, TOV0 // Limpiar bandera de overflow
	LDI		R16, 251 // INTERRUPCION CADA 5 ms
	OUT		TCNT0, R16
	MOV		R16, LED_CENTRAL
	CPI		R16, 30
	BRNE	ALTERNAR_3
	LDI		R16, 0x00
	MOV		LED_CENTRAL, R16
	SBI		PINB, 3
ALTERNAR_3:
	INC		LED_CENTRAL
    LSL		PUNTERO_DISPLAYS         //Desplazar a la izquierda (0001 ? 0010 ? 0100 ? 1000)
	MOV		R16, PUNTERO_DISPLAYS
    CPI		R16, 0b00010000			//Si el valor es mayor a 1000, reiniciar
    BRNE	CONTINUAR1_3
    LDI		R16, 0b00000001			//Reiniciar a 0001
	MOV		PUNTERO_DISPLAYS, R16
CONTINUAR1_3:
    OUT		PORTC, PUNTERO_DISPLAYS	//Enviar el valor a PORTC
	MOV		R16, PUNTERO_DISPLAYS
	CPI		R16, 0b00000001
	BREQ	DISPLAY4_3
	CPI		R16, 0b00000010
	BREQ	DISPLAY3_3
	CPI		R16, 0b00000100
	BREQ	DISPLAY2_3
	CPI		R16, 0b00001000
	BREQ	DISPLAY1_3
DISPLAY1_3:
	OUT		PORTD, UNIDADES_MINUTOS
	RETI
DISPLAY2_3:
	OUT		PORTD, DECENAS_MINUTOS
	RETI
DISPLAY3_3:
	OUT		PORTD, UNIDADES_HORAS
	RETI
DISPLAY4_3:
	OUT		PORTD, DECENAS_HORAS
	RETI
//===================================================================================================================================
//===================================================================================================================================
//MOSTRAR FECHA CONFIGURACION
//===================================================================================================================================
//===================================================================================================================================
MOSTRAR_FECHA_CONFIG:
    SBI		TIFR0, TOV0 // Limpiar bandera de overflow
	LDI		R16, 251 // INTERRUPCION CADA 5 ms
	OUT		TCNT0, R16
	MOV		R16, LED_CENTRAL
	CPI		R16, 30
	BRNE	ALTERNAR1_4
	LDI		R16, 0x00
	MOV		LED_CENTRAL, R16
	SBI		PINB, 3
	SBI		PINB, 5
ALTERNAR1_4:
	INC		LED_CENTRAL
    LSL		PUNTERO_DISPLAYS         //Desplazar a la izquierda (0001 ? 0010 ? 0100 ? 1000)
	MOV		R16, PUNTERO_DISPLAYS
    CPI		R16, 0b00010000			//Si el valor es mayor a 1000, reiniciar
    BRNE	CONTINUAR1_4
    LDI		R16, 0b00000001			//Reiniciar a 0001
	MOV		PUNTERO_DISPLAYS, R16
CONTINUAR1_4:
    OUT		PORTC, PUNTERO_DISPLAYS		//Enviar el valor a PORTC
	MOV		R16, PUNTERO_DISPLAYS
	CPI		R16, 0b00000001
	BREQ	DISPLAY4_4
	CPI		R16, 0b00000010
	BREQ	DISPLAY3_4
	CPI		R16, 0b00000100
	BREQ	DISPLAY2_4
	CPI		R16, 0b00001000
	BREQ	DISPLAY1_4
DISPLAY1_4:
	OUT		PORTD, UNIDADES_MESES
	RETI
DISPLAY2_4:
	OUT		PORTD, DECENAS_MESES
	RETI
DISPLAY3_4:
	OUT		PORTD, UNIDADES_DIAS
	RETI
DISPLAY4_4:
	OUT		PORTD, DECENAS_DIAS
	RETI
//===================================================================================================================================


.def CONTADOR_MINUTOS1 = R2
.def CONTADOR_HORAS1 = R3
.def ACTUALIZAR_U_HORAS = R4
.def LEC_BOTON = R5
.def PUNTERO_DISPLAYS = R6
.def LED_CENTRAL = R7
.def CONTADOR_DIAS1 = R8
.def CONTADOR_MESES = R9
.def CONTADOR_DIAS2 = R10
.def CONTADOR_MINUTOS = R11
.def CONTADOR_MESES1 = R12
.def CONTADOR_ALARMA1 = R13
.def CONTADOR_ALARMA2 = R14
.def CONTADOR_ALARMA3 = R15
.def CONTADOR_ALARMA4 = R1
.def UNIDADES_MINUTOS = R17
.def DECENAS_MINUTOS = R18
.def UNIDADES_HORAS = R19
.def DECENAS_HORAS = R20
.def UNIDADES_DIAS = R21
.def DECENAS_DIAS = R22
.def UNIDADES_MESES = R23
.def DECENAS_MESES = R24
.def UNIDADES_ALARMA_M = R25
.def DECENAS_ALARMA_M = R26
.def UNIDADES_ALARMA_H = R27
.def DECENAS_ALARMA_H = R28
.def MODOS = R29
LISTA: .db 0xC0, 0xF9, 0xA4, 0xB0, 0x99, 0x92, 0x82, 0xF8, 0x80, 0x90
