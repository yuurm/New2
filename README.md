import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import static org.apache.spark.sql.functions.collect_list;

public class JoinDatasetArrayColumn {

    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
                .appName("JoinDatasetArrayColumn")
                .getOrCreate();

        // Загрузка данных из Hive таблиц в датасеты
        Dataset<Row> dataset1 = spark.sql("SELECT * FROM dataset1_table");
        Dataset<Row> dataset2 = spark.sql("SELECT * FROM dataset2_table");

        // Выполнение джойна двух датасетов
        Dataset<Row> joinedDataset = dataset1.join(dataset2, "common_column");

        // Сохранение всех колонок из первого датасета
        String[] columns = dataset1.columns();
        Dataset<Row> finalDataset = joinedDataset.selectExpr(columns)
                .withColumn("array_column", collect_list(joinedDataset.col("column_to_convert")));

        // Вывод структуры созданного датасета
        finalDataset.printSchema();

        spark.stop();
    }
}
