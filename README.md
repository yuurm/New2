import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.expressions.Window;
import org.apache.spark.sql.expressions.WindowSpec;
import static org.apache.spark.sql.functions.*;
import org.apache.spark.sql.expressions.WindowSpec;

public class Main {
    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
                .appName("PartitionAndFindPreviousRecord")
                .getOrCreate();

        // Загрузка датасета
        Dataset<Row> dataset = spark.read().csv("путь_к_вашему_файлу.csv");

        // Определение окна для партиционирования
        WindowSpec windowSpec = Window.partitionBy("деп_м", "деп_н").orderBy(col("целочисленная_колонка"));

        // Добавление колонки с предыдущим значением
        Dataset<Row> result = dataset.withColumn("предыдущее_значение",
                lag(col("целочисленная_колонка"), 1).over(windowSpec));

        // Вывод результата
        result.show();

        spark.stop();
    }
}
