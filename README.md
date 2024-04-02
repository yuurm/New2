import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

import java.util.List;

public class DepositOperationJoin {
    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
                .appName("DepositOperationJoin")
                .enableHiveSupport()
                .getOrCreate();

        // Загрузка данных из Hive таблиц в Spark датасеты
        Dataset<Row> depositsDF = spark.sql("SELECT * FROM deposits_table");
        Dataset<Row> operationsDF = spark.sql("SELECT account_id, collect_list(operation) as operations FROM operations_table GROUP BY account_id");

        // Джойн датасетов по account_id
        Dataset<Row> joinedDF = depositsDF.join(operationsDF, "account_id");

        // Преобразование результирующего депозита в массив Java объектов
        List<DepositWithOperations> depositList = joinedDF.as(Encoders.bean(DepositWithOperations.class)).collectAsList();

        // Вывод результатов
        for (DepositWithOperations deposit : depositList) {
            System.out.println(deposit);
        }

        spark.stop();
    }

    public static class DepositWithOperations {
        private int account_id;
        private List<String> operations;

        public int getAccount_id() {
            return account_id;
        }

        public void setAccount_id(int account_id) {
            this.account_id = account_id;
        }

        public List<String> getOperations() {
            return operations;
        }

        public void setOperations(List<String> operations) {
            this.operations = operations;
        }

        @Override
        public String toString() {
            return "DepositWithOperations{" +
                    "account_id=" + account_id +
                    ", operations=" + operations +
                    '}';
        }
    }
}
