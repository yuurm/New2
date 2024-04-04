Для переписывания кода на использование мэппартишнс в Apache Spark, вы можете использовать метод mapPartitions для обработки каждой партиции данных. Вот пример кода, который демонстрирует использование мэппартишнс:

import org.apache.spark.api.java.function.MapPartitionsFunction;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

public class Main {
    public static void main(String[] args) {
        SparkSession spark = SparkSession
                .builder()
                .appName("Java Spark Example")
                .getOrCreate();

        // Читаем исходный датасет
        Dataset<Row> dataset = spark.read().csv("path/to/dataset.csv");

        // Применяем mapPartitions к датасету
        Dataset<Result> resultDataset = dataset.mapPartitions((MapPartitionsFunction<Row, Result>) iterator -> {
            // Создаем экземпляр калькулятора
            Calculator calculator = new Calculator();

            // Создаем объект Input для каждой партиции
            Input input = calculator.createInput();

            // Вычисляем результат для каждой строки в партиции
            List<Result> results = new ArrayList<>();
            while (iterator.hasNext()) {
                Row row = iterator.next();
                Result result = calculator.getCalc().calcWithRes(input);
                results.add(result);
            }

            return results.iterator();
        }, Encoders.bean(Result.class));

        resultDataset.show();

        spark.stop();
    }
}

class Calculator {
    public Input createInput() {
        // Логика создания объекта Input
        return new Input();
    }

    public Calc getCalc() {
        // Логика получения объекта Calc
        return new Calc();
    }
}

class Input {
    // Поля и методы объекта Input
}

class Calc {
    public Result calcWithRes(Input input) {
        // Логика вычисления результата
        return new Result();
    }
}

class Result {
    // Поля и методы объекта Result
}


В этом примере мы используем метод mapPartitions для обработки каждой партиции данных в датасете. Внутри функции mapPartitions мы создаем объект Calculator, создаем объект Input для каждой партиции и вычисляем результат для каждой строки в партиции с помощью метода calcWithRes объекта Calc. Результаты сохраняются в список и возвращаются как итератор.

Пожалуйста, убедитесь, что у вас установлена библиотека Apache Spark и правильно сконфигурированы все необходимые зависимости перед запуском этого кода.
