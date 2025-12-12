import java.util.*;

public class Converter {
    // Справочники коэффициентов перевода (относительно базовой единицы)
    private static final Map<String, Map<String, Double>> UNITS = new HashMap<>();

    static {
        // Объём
        Map<String, Double> volume = new HashMap<>();
        volume.put("л", 1.0);
        volume.put("мл", 0.001);
        volume.put("м³", 1000.0);
        volume.put("см³", 0.001);
        volume.put("галлон", 3.78541);
        volume.put("пинта", 0.473176);
        UNITS.put("объём", volume);

        // Длина
        Map<String, Double> length = new HashMap<>();
        length.put("м", 1.0);
        length.put("см", 0.01);
        length.put("мм", 0.001);
        length.put("км", 1000.0);
        length.put("дюйм", 0.0254);
        length.put("фут", 0.3048);
        length.put("миля", 1609.34);
        UNITS.put("длина", length);

        // Масса
        Map<String, Double> mass = new HashMap<>();
        mass.put("кг", 1.0);
        mass.put("г", 0.001);
        mass.put("мг", 0.000001);
        mass.put("т", 1000.0);
        mass.put("фунт", 0.453592);
        mass.put("унция", 0.0283495);
        UNITS.put("масса", mass);

        // Время
        Map<String, Double> time = new HashMap<>();
        time.put("с", 1.0);
        time.put("мин", 60.0);
        time.put("ч", 3600.0);
        time.put("день", 86400.0);
        time.put("неделя", 604800.0);
        time.put("год", 31536000.0); // 365 дней
        UNITS.put("время", time);

        // Площадь
        Map<String, Double> area = new HashMap<>();
        area.put("м²", 1.0);
        area.put("см²", 0.0001);
        area.put("км²", 1000000.0);
        area.put("га", 10000.0);
        area.put("акр", 4046.86);
        UNITS.put("площадь", area);
    }

    /** Конвертация температур между °C, °F и K */
    private static double convertTemperature(double value, String fromUnit, String toUnit) throws IllegalArgumentException {
        double celsius;

        // Переводим в Цельсий
        if (fromUnit.equals("°C")) {
            celsius = value;
        } else if (fromUnit.equals("°F")) {
            celsius = (value - 32) * 5 / 9;
        } else if (fromUnit.equals("K")) {
            celsius = value - 273.15;
        } else {
            throw new IllegalArgumentException("Неизвестная единица температуры: " + fromUnit);
        }

        // Переводим из Цельсия
        if (toUnit.equals("°C")) {
            return celsius;
        } else if (toUnit.equals("°F")) {
            return celsius * 9 / 5 + 32;
        } else if (toUnit.equals("K")) {
            return celsius + 273.15;
        } else {
            throw new IllegalArgumentException("Неизвестная единица температуры: " + toUnit);
        }
    }

    /** Отображает главное меню */
    private static void showMenu() {
        System.out.println("\n========================================");
        System.out.println("      КОНВЕРТЕР ЕДИНИЦ ИЗМЕРЕНИЯ");
        System.out.println("========================================");

        int i = 1;
        for (String category : UNITS.keySet()) {
            System.out.printf("%d. %s\n", i++, category.toUpperCase());
        }
        System.out.println("6. ТЕМПЕРАТУРА");
        System.out.println("0. ВЫХОД");
        System.out.println("----------------------------------------");
    }

    /** Получает и обрабатывает ввод пользователя */
    private static Object[] getUserInput(Scanner scanner) {
        try {
            System.out.print("Выберите категорию (0–6): ");
            String choiceStr = scanner.nextLine().trim();
            int choice = Integer.parseInt(choiceStr);

            if (choice == 0) return null;

            String category;
            if (choice == 6) {
                category = "температура";
            } else if (choice >= 1 && choice <= 5) {
                List<String> categories = new ArrayList<>(UNITS.keySet());
                category = categories.get(choice - 1);
            } else {
                System.out.println("Выберите число от 1 до 6!");
                return new Object[]{null, null, null, null};
            }

            System.out.print("Введите значение: ");
            double value = Double.parseDouble(scanner.nextLine().trim());

            System.out.println("\nДоступные единицы для " + category + ":");
            if (category.equals("температура")) {
                System.out.println("°C, °F, K");
            } else {
                System.out.println(String.join(", ", UNITS.get(category).keySet()));
            }

            System.out.print("Из единицы: ");
            String fromUnit = scanner.nextLine().trim();

            System.out.print("В единицу: ");
            String toUnit = scanner.nextLine().trim();

            return new Object[]{category, value, fromUnit, toUnit};

        } catch (NumberFormatException e) {
            System.out.println("Ошибка: введите числовое значение!");
        } catch (Exception e) {
            System.out.println("Ошибка ввода: " + e.getMessage());
        }
        return new Object[]{null, null, null, null};
    }

    /** Выполняет конвертацию */
    private static Double convert(String category, double value, String fromUnit, String toUnit) {
        try {
            if (category.equals("температура")) {
                return convertTemperature(value, fromUnit, toUnit);
            } else {
                Map<String, Double> unitsMap = UNITS.get(category);
                if (!unitsMap.containsKey(fromUnit)) {
                    throw new IllegalArgumentException("Единица '" + fromUnit + "' не найдена для '" + category + "'");
                }
                if (!unitsMap.containsKey(toUnit)) {
                    throw new IllegalArgumentException("Единица '" + toUnit + "' не найдена для '" + category + "'");
                }

                double baseValue = value * unitsMap.get(fromUnit);
                return baseValue / unitsMap.get(toUnit);
            }
        } catch (Exception e) {
            System.out.println("Ошибка конвертации: " + e.getMessage());
            return null;
        }
    }

    /** Основная функция программы */
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Добро пожаловать в конвертер единиц измерения!");

        while (true) {
            showMenu();
            Object[] input = getUserInput(scanner);

            if (input == null) { // Пользователь выбрал выход
                System.out.println("До свидания! 👋");
                break;
            }

            String category = (String) input[0];
            double value = (Double) input[1];
            String fromUnit = (String) input[2];
            String toUnit = (String) input[3];

            Double result = convert(category, value, fromUnit, toUnit);

                        if (result != null) {
                System.out.println("\n✅ Результат:");
                System.out.printf("%.6g %s = %.6g %s\n", value, fromUnit, result, toUnit);
            }

            System.out.println("----------------------------------------");
            System.out.print("Продолжить? (y/n): ");
            String continueChoice = scanner.nextLine().trim().toLowerCase();
            if (continueChoice.equals("n") || continueChoice.equals("нет")) {
                System.out.println("До свидания! 👋");
                break;
            }
        }

        scanner.close();
    }
}
