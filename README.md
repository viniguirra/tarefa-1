# tarefa-1
#include <stdio.h>
#include "pico/stdlib.h"
#include "hardware/gpio.h"
#include "ws2812.h"

#define LED_R 11
#define LED_G 12
#define LED_B 13
#define BUTTON_A 5
#define BUTTON_B 6
#define WS2812_PIN 7
#define NUM_LEDS 25

volatile int numero_atual = 0;

// Mapeamento dos números para a matriz 5x5
const uint8_t numeros_ws2812[10][25] = {
    {1,1,1,1,1, 1,0,0,0,1, 1,0,0,0,1, 1,0,0,0,1, 1,1,1,1,1},
    {0,0,1,0,0, 0,1,1,0,0, 1,0,1,0,0, 0,0,1,0,0, 1,1,1,1,1},
    // Adicionar representações para 2-9...
};

void exibir_numero(int numero) {
    for (int i = 0; i < NUM_LEDS; i++) {
        set_led(i, numeros_ws2812[numero][i] ? 255 : 0, numeros_ws2812[numero][i] ? 255 : 0, numeros_ws2812[numero][i] ? 255 : 0);
    }
    show();
}

void debounce(uint gpio, uint32_t events) {
    sleep_ms(50);
}

void incrementar_numero(uint gpio, uint32_t events) {
    debounce(gpio, events);
    numero_atual = (numero_atual + 1) % 10;
    exibir_numero(numero_atual);
}

void decrementar_numero(uint gpio, uint32_t events) {
    debounce(gpio, events);
    numero_atual = (numero_atual - 1) % 10;
    exibir_numero(numero_atual);
}

int main() {
    stdio_init_all();
    gpio_init(LED_R);
    gpio_set_dir(LED_R, GPIO_OUT);
    gpio_init(BUTTON_A);
    gpio_set_dir(BUTTON_A, GPIO_IN);
    gpio_pull_up(BUTTON_A);
    gpio_init(BUTTON_B);
    gpio_set_dir(BUTTON_B, GPIO_IN);
    gpio_pull_up(BUTTON_B);

    gpio_set_irq_enabled_with_callback(BUTTON_A, GPIO_IRQ_EDGE_FALL, true, &incrementar_numero);
    gpio_set_irq_enabled_with_callback(BUTTON_B, GPIO_IRQ_EDGE_FALL, true, &decrementar_numero);

    while (1) {
        gpio_put(LED_R, 1);
        sleep_ms(100);
        gpio_put(LED_R, 0);
        sleep_ms(100);
    }
}
