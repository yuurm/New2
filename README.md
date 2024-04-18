import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import static org.apache.spark.sql.functions.*;

Dataset<Row> yourDataset = // ваш датасет

yourDataset = yourDataset.withColumn("new_date_column",
    when(col("date_of_payment").isNotNull().and(col("date_of_opening").isNull().or(col("date_of_opening").lt(col("date_of_payment"))).and(col("start_date").isNull().or(col("start_date").lt(col("date_of_payment")))),
        col("date_of_payment"))
    .when(col("date_of_opening").isNotNull().and(col("date_of_payment").isNull().or(col("date_of_payment").lt(col("date_of_opening"))).and(col("start_date").isNull().or(col("start_date").lt(col("date_of_opening")))),
        col("date_of_opening"))
    .otherwise(col("start_date"))
    .cast("date")
);

yourDataset.show();
