import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

public class CheckColumnNames {

    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
                .appName("Check Column Names")
                .getOrCreate();

        // Загрузка вашего датасета
        Dataset<Row> df = spark.read().format("csv").option("header", "true").load("путь_к_вашему_файлу.csv");

        // Получение списка всех названий колонок
        String[] allColumns = df.columns();

        // Названия колонок, которые вы хотите проверить
        String[] requiredColumns = {"column1", "column2", "arrayColumn.field1", "arrayColumn.field2"};

        // Проверка наличия каждого из требуемых названий колонок
        for (String column : requiredColumns) {
            if (!containsColumn(allColumns, column)) {
                System.out.println("Отсутствует требуемая колонка: " + column);
            }
        }
    }

    private static boolean containsColumn(String[] columns, String columnName) {
        for (String column : columns) {
            if (column.equals(columnName)) {
                return true;
            }
        }
        return false;
    }
}
