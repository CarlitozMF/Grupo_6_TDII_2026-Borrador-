/* USER CODE BEGIN Header */
/**
 ******************************************************************************
 * @file           : main.c
 * @brief          : Main program body
 ******************************************************************************
 * @attention
 *
 * Copyright (c) 2026 STMicroelectronics.
 * All rights reserved.
 *
 * This software is licensed under terms that can be found in the LICENSE file
 * in the root directory of this software component.
 * If no LICENSE file comes with this software, it is provided AS-IS.
 *
 ******************************************************************************
 */
/* USER CODE END Header */
/* Includes ------------------------------------------------------------------*/
#include "main.h"

/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */

#include "stdbool.h"
#include "driver_led.h"
#include "driver_led_pal.h"
#include "driver_time.h"
#include "driver_boton.h"
#include "driver_boton_pal.h"

/* USER CODE END Includes */

/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */

/** * @brief Definición de estados para la MEF.
 */
typedef enum{
	Secuencia_1 = 0,
	Secuencia_2,
	Secuencia_3,
	Secuencia_4
} Estado_t;


/**
 * @brief Definición de estados para la MEF de Secuencia 1.
 */
typedef enum{
	Estado_Encendido = 0,
	Estado_Apagado
} Estado_Sec1_t;

/* USER CODE END PTD */

/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */

/* USER CODE END PD */

/* Private macro -------------------------------------------------------------*/
/* USER CODE BEGIN PM */

/* USER CODE END PM */

/* Private variables ---------------------------------------------------------*/
UART_HandleTypeDef huart3;

/* USER CODE BEGIN PV */

#define INDICE_BTN_USER 0

// Tablas físicas asociadas a las estructuras PAL de tus drivers propios
static const BOTON_Hardware_t tabla_botones_hw[] = {
    [INDICE_BTN_USER] = {USER_BTN_GPIO_Port, USER_BTN_Pin, BOTON_ACTIVO_ALTO}
};
static const uint8_t cant_botones = sizeof(tabla_botones_hw) / sizeof(tabla_botones_hw[0]);

static const LED_Hardware_t tabla_leds_hardware[] = {
    {LD1_GPIO_Port, LD1_Pin},
    {LD2_GPIO_Port, LD2_Pin},
    {LD3_GPIO_Port, LD3_Pin}
};
static const uint8_t total_leds = sizeof(tabla_leds_hardware) / sizeof(tabla_leds_hardware[0]);

// Tiempos de alternancia del algoritmo clásico
static const uint32_t alternancia_sec1 = 150;
static const uint32_t alternancia_sec2 = 300;

// Variables de control de las secuencias
static int8_t indice_led = 0;
static Estado_t estado_secuencia = Secuencia_1;     // MEF Principal
static Estado_Sec1_t modo_sec1 = Estado_Encendido;   // MEF Secundaria de la Secuencia 1
static uint32_t contador_ciclos_sec3 = 0;           // Reemplaza los divisores imprecisos
static bool estado_parpadeo_sec4 = false;

/* USER CODE END PV */

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_USART3_UART_Init(void);
/* USER CODE BEGIN PFP */

/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */

/* USER CODE END 0 */

/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{

  /* USER CODE BEGIN 1 */

  /* USER CODE END 1 */

  /* MCU Configuration--------------------------------------------------------*/

  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* USER CODE BEGIN Init */

  /* USER CODE END Init */

  /* Configure the system clock */
  SystemClock_Config();

  /* USER CODE BEGIN SysInit */

  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_USART3_UART_Init();
  /* USER CODE BEGIN 2 */

  LED_Init(tabla_leds_hardware, total_leds);
  BOTON_Init(tabla_botones_hw, cant_botones);

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
	while (1)
	{
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */

		/* 1. Detección de flanco (Usa el driver genérico del botón afuera del switch) */
		        if (BOTON_DetectarFlancoPresionado(INDICE_BTN_USER)) {

		            // Apagado de seguridad genérico usando el driver antes de cambiar de estado
		            for (uint8_t i = 0; i < LED_GetCount(); i++) {
		                LED_Write(i, false);
		            }

		            // Transición circular de la MEF Principal
		            estado_secuencia++;
		            if (estado_secuencia > Secuencia_4) {
		                estado_secuencia = Secuencia_1;
		            }

		            // Reinicio de variables de estado locales en español
		            indice_led = 0;
		            modo_sec1 = Estado_Encendido;
		            contador_ciclos_sec3 = 0;
		            estado_parpadeo_sec4 = false;
		        }

		        /* 2. Máquina de Estados (MEF) Principal */
		        switch (estado_secuencia) {

		        case Secuencia_1:
		            /* Secuencia de avance unidireccional clásica */
		            switch (modo_sec1) {
		            case Estado_Encendido:
		                LED_Write(indice_led, true);
		                TIME_DelayMS(alternancia_sec1);
		                modo_sec1 = Estado_Apagado;
		                break;

		            case Estado_Apagado:
		                LED_Write(indice_led, false);
		                TIME_DelayMS(alternancia_sec1);
		                modo_sec1 = Estado_Encendido;

		                indice_led++;
		                if (indice_led >= LED_GetCount()) {
		                    indice_led = 0;
		                }
		                break;

		            default:
		                modo_sec1 = Estado_Encendido;
		                indice_led = 0;
		                break;
		            }
		            break;

		        case Secuencia_2:
		            /* Parpadeo simultáneo de todos los LEDs de la tabla */
		            for (uint8_t j = 0; j < LED_GetCount(); j++) {
		                LED_Write(j, true);
		            }
		            TIME_DelayMS(alternancia_sec2);

		            for (uint8_t j = 0; j < LED_GetCount(); j++) {
		                LED_Write(j, false);
		            }
		            TIME_DelayMS(alternancia_sec2);
		            break;

		        case Secuencia_3:
		            /* Parpadeo asincrónico: Base bloqueante controlada de 100ms */
		            TIME_DelayMS(100);
		            contador_ciclos_sec3++; // Cada ciclo representa exactamente 100ms reales

		            // LED 1: Cambia cada 1 ciclo (100ms)
		            if (contador_ciclos_sec3 % 1 == 0) {
		                LED_Toggle(0);
		            }
		            // LED 2: Cambia cada 3 ciclos (300ms)
		            if (contador_ciclos_sec3 % 3 == 0) {
		                LED_Toggle(1);
		            }
		            // LED 3: Cambia cada 6 ciclos (600ms)
		            if (contador_ciclos_sec3 % 6 == 0) {
		                LED_Toggle(2);
		            }
		            break;

		        case Secuencia_4:
		            /* Parpadeo cruzado simétrico cada 300ms */
		            estado_parpadeo_sec4 = !estado_parpadeo_sec4;

		            // LED 1 y 3 en fase / LED 2 en contrafase
		            LED_Write(0, estado_parpadeo_sec4);
		            LED_Write(2, estado_parpadeo_sec4);
		            LED_Write(1, !estado_parpadeo_sec4);

		            TIME_DelayMS(300); // Retardo controlado estable
		            break;

		        default:
		            // Recuperación ante fallas y reinicio general seguro
		            for (uint8_t j = 0; j < LED_GetCount(); j++) {
		                LED_Write(j, false);
		            }
		            estado_secuencia = Secuencia_1;
		            modo_sec1 = Estado_Encendido;
		            indice_led = 0;
		            contador_ciclos_sec3 = 0;
		            estado_parpadeo_sec4 = false;
		            break;
		        }
	}
  /* USER CODE END 3 */
}

/**
  * @brief System Clock Configuration
  * @retval None
  */
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  /** Configure the main internal regulator output voltage
  */
  __HAL_RCC_PWR_CLK_ENABLE();
  __HAL_PWR_VOLTAGESCALING_CONFIG(PWR_REGULATOR_VOLTAGE_SCALE1);

  /** Initializes the RCC Oscillators according to the specified parameters
  * in the RCC_OscInitTypeDef structure.
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
  RCC_OscInitStruct.HSEState = RCC_HSE_ON;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
  RCC_OscInitStruct.PLL.PLLM = 4;
  RCC_OscInitStruct.PLL.PLLN = 180;
  RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV2;
  RCC_OscInitStruct.PLL.PLLQ = 7;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  /** Activate the Over-Drive mode
  */
  if (HAL_PWREx_EnableOverDrive() != HAL_OK)
  {
    Error_Handler();
  }

  /** Initializes the CPU, AHB and APB buses clocks
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV4;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV2;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_5) != HAL_OK)
  {
    Error_Handler();
  }
}

/**
  * @brief USART3 Initialization Function
  * @param None
  * @retval None
  */
static void MX_USART3_UART_Init(void)
{

  /* USER CODE BEGIN USART3_Init 0 */

  /* USER CODE END USART3_Init 0 */

  /* USER CODE BEGIN USART3_Init 1 */

  /* USER CODE END USART3_Init 1 */
  huart3.Instance = USART3;
  huart3.Init.BaudRate = 115200;
  huart3.Init.WordLength = UART_WORDLENGTH_8B;
  huart3.Init.StopBits = UART_STOPBITS_1;
  huart3.Init.Parity = UART_PARITY_NONE;
  huart3.Init.Mode = UART_MODE_TX_RX;
  huart3.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart3.Init.OverSampling = UART_OVERSAMPLING_16;
  if (HAL_UART_Init(&huart3) != HAL_OK)
  {
    Error_Handler();
  }
  /* USER CODE BEGIN USART3_Init 2 */

  /* USER CODE END USART3_Init 2 */

}

/**
  * @brief GPIO Initialization Function
  * @param None
  * @retval None
  */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};
  /* USER CODE BEGIN MX_GPIO_Init_1 */

  /* USER CODE END MX_GPIO_Init_1 */

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOC_CLK_ENABLE();
  __HAL_RCC_GPIOH_CLK_ENABLE();
  __HAL_RCC_GPIOB_CLK_ENABLE();
  __HAL_RCC_GPIOD_CLK_ENABLE();
  __HAL_RCC_GPIOG_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();

  /*Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(GPIOB, LD1_Pin|LD3_Pin|LD2_Pin, GPIO_PIN_RESET);

  /*Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(USB_PowerSwitchOn_GPIO_Port, USB_PowerSwitchOn_Pin, GPIO_PIN_RESET);

  /*Configure GPIO pin : USER_BTN_Pin */
  GPIO_InitStruct.Pin = USER_BTN_Pin;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  HAL_GPIO_Init(USER_BTN_GPIO_Port, &GPIO_InitStruct);

  /*Configure GPIO pin : PC4 */
  GPIO_InitStruct.Pin = GPIO_PIN_4;
  GPIO_InitStruct.Mode = GPIO_MODE_IT_RISING;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  HAL_GPIO_Init(GPIOC, &GPIO_InitStruct);

  /*Configure GPIO pins : LD1_Pin LD3_Pin LD2_Pin */
  GPIO_InitStruct.Pin = LD1_Pin|LD3_Pin|LD2_Pin;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);

  /*Configure GPIO pin : USB_PowerSwitchOn_Pin */
  GPIO_InitStruct.Pin = USB_PowerSwitchOn_Pin;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(USB_PowerSwitchOn_GPIO_Port, &GPIO_InitStruct);

  /*Configure GPIO pin : USB_OverCurrent_Pin */
  GPIO_InitStruct.Pin = USB_OverCurrent_Pin;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  HAL_GPIO_Init(USB_OverCurrent_GPIO_Port, &GPIO_InitStruct);

  /* USER CODE BEGIN MX_GPIO_Init_2 */

  /* USER CODE END MX_GPIO_Init_2 */
}

/* USER CODE BEGIN 4 */

/* USER CODE END 4 */

/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
	/* User can add his own implementation to report the HAL error return state */
	__disable_irq();
	while (1)
	{
	}
  /* USER CODE END Error_Handler_Debug */
}
#ifdef USE_FULL_ASSERT
/**
  * @brief  Reports the name of the source file and the source line number
  *         where the assert_param error has occurred.
  * @param  file: pointer to the source file name
  * @param  line: assert_param error line source number
  * @retval None
  */
void assert_failed(uint8_t *file, uint32_t line)
{
  /* USER CODE BEGIN 6 */
	/* User can add his own implementation to report the file name and line number,
     ex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */
