import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import static org.apache.spark.sql.functions.col;
import static org.apache.spark.sql.functions.explode;

public class ArrayStructExplode {
    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
                .appName("ArrayStructExplode")
                .master("local")
                .getOrCreate();

        // Пример данных
        Dataset<Row> data = spark.createDataFrame(
                Arrays.asList(
                        RowFactory.create(Arrays.asList(RowFactory.create("John", 25), RowFactory.create("Alice", 30))),
                        RowFactory.create(Arrays.asList(RowFactory.create("Bob", 28), RowFactory.create("Eve", 35)))
                ),
                DataTypes.createStructType(Arrays.asList(
                        DataTypes.createStructField("persons", DataTypes.createArrayType(
                                DataTypes.createStructType(Arrays.asList(
                                        DataTypes.createStructField("name", DataTypes.StringType, false),
                                        DataTypes.createStructField("age", DataTypes.IntegerType, false)
                                ))
                        ), false)
                ))
        );

        // Конвертация массива структур в строки
        Dataset<Row> explodedData = data.select(explode(col("persons")).as("person"));

        explodedData.show();
    }
}
