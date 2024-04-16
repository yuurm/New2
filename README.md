import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import static org.apache.spark.sql.functions.*;

public class JoinDatasets {
    public static void main(String[] args) {
        // Создаем SparkSession
        SparkSession spark = SparkSession.builder()
                .appName("Join Datasets")
                .master("local")
                .getOrCreate();

        // Создаем первый датасет
        Dataset<Row> dataset1 = ...; // Ваш первый датасет

        // Создаем второй датасет
        Dataset<Row> dataset2 = ...; // Ваш второй датасет

        // Определяем условие для джойна
        Dataset<Row> joinedDataset = dataset1.join(dataset2,
                col("integerColumn").between(col("startDate").cast("int"), col("actionDate").cast("int"))
                        .or(col("integerColumn").between(col("startDate"), col("actionDate"))),
                "inner");

        // Показываем результат джойна
        joinedDataset.show();

        // Остановка SparkSession
        spark.stop();
    }
}
