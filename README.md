#include "pico/stdlib.h"
#include "hardware/adc.h"
#include <stdio.h>

const uint LED_PURPLE_PIN = 2;
const uint LED_CYAN_PIN   = 18;
const uint LED_BLUE_PIN   = 16;
const uint LED_GREEN_PIN  = 15;
const uint LED_WHITE_PIN  = 1;

const uint JOYSTICK_X_ADC_PIN = 27;
const uint JOYSTICK_Y_ADC_PIN = 26;
const uint JOYSTICK_SW_PIN    = 17;

const uint16_t ADC_LOW_THRESHOLD  = 38;
const uint16_t ADC_HIGH_THRESHOLD = 3749;

int main() {
    stdio_init_all();
    printf("--- Raspberry Pi Pico Joystick & LED Control (Python-like Logic) ---\n");

    gpio_init(LED_PURPLE_PIN); gpio_set_dir(LED_PURPLE_PIN, GPIO_OUT);
    gpio_init(LED_CYAN_PIN);   gpio_set_dir(LED_CYAN_PIN, GPIO_OUT);
    gpio_init(LED_BLUE_PIN);   gpio_set_dir(LED_BLUE_PIN, GPIO_OUT);
    gpio_init(LED_GREEN_PIN);  gpio_set_dir(LED_GREEN_PIN, GPIO_OUT);
    gpio_init(LED_WHITE_PIN);  gpio_set_dir(LED_WHITE_PIN, GPIO_OUT);
    printf("All LED pins initialized as outputs.\n");

    adc_init();
    adc_gpio_init(JOYSTICK_X_ADC_PIN);
    adc_gpio_init(JOYSTICK_Y_ADC_PIN);
    printf("ADC for Joystick X (GP%d/ADC1) and Y (GP%d/ADC0) initialized.\n", JOYSTICK_X_ADC_PIN, JOYSTICK_Y_ADC_PIN);

    gpio_init(JOYSTICK_SW_PIN);
    gpio_set_dir(JOYSTICK_SW_PIN, GPIO_IN);
    gpio_pull_up(JOYSTICK_SW_PIN);
    printf("Joystick Button (GP%d) initialized with pull-up.\n", JOYSTICK_SW_PIN);

    while (true) {
        adc_select_input(1);
        uint16_t xValue = adc_read();

        adc_select_input(0);
        uint16_t yValue = adc_read();

        int buttonValue = gpio_get(JOYSTICK_SW_PIN);

        const char* xStatus = "middle";
        const char* yStatus = "middle";
        const char* buttonStatus = "not pressed";

        gpio_put(LED_PURPLE_PIN, 0);
        gpio_put(LED_WHITE_PIN, 0);
        gpio_put(LED_GREEN_PIN, 0);
        gpio_put(LED_BLUE_PIN, 0);
        gpio_put(LED_CYAN_PIN, 1);

        if (buttonValue == 0) {
            buttonStatus = "pressed";
            gpio_put(LED_CYAN_PIN, 0);
        }

        if (xValue <= ADC_LOW_THRESHOLD) {
            xStatus = "left";
            gpio_put(LED_PURPLE_PIN, 1);
            gpio_put(LED_CYAN_PIN, 0);
        } else if (xValue >= ADC_HIGH_THRESHOLD) {
            xStatus = "right";
            gpio_put(LED_BLUE_PIN, 1);
            gpio_put(LED_CYAN_PIN, 0);
        }

        if (yValue <= ADC_LOW_THRESHOLD) {
            yStatus = "up";
            gpio_put(LED_GREEN_PIN, 1);
            gpio_put(LED_CYAN_PIN, 0);
        } else if (yValue >= ADC_HIGH_THRESHOLD) {
            yStatus = "down";
            gpio_put(LED_WHITE_PIN, 1);
            gpio_put(LED_CYAN_PIN, 0);
        }

        printf("X: %s (%u), Y: %s (%u), button status: %s\n",
               xStatus, xValue, yStatus, yValue, buttonStatus);

        sleep_ms(200);
    }

    return 0;
}
