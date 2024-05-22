In Apache Spark, there isn't a built-in auto-increment feature like in traditional databases. However, you can simulate auto-increment functionality by creating a custom sequence and using it to generate unique IDs for your primary key field. Here's an example of how you can achieve this in Spark using a custom function:

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.Encoders;
import org.apache.spark.sql.functions;

import java.util.Arrays;
import java.util.List;

public class AutoIncrementPrimaryKey {

    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
                .appName("AutoIncrementPrimaryKey")
                .master("local[*]")
                .getOrCreate();

        // Sample data of type Student
        List<Student> students = Arrays.asList(
                new Student("Alice"),
                new Student("Bob")
        );

        // Creating a Dataset of Students
        Dataset<Student> studentDataset = spark.createDataset(students, Encoders.bean(Student.class));

        // Add an auto-increment primary key column
        Dataset<Row> indexedStudentDataset = addAutoIncrementPrimaryKey(studentDataset);

        indexedStudentDataset.show();
    }

    // Function to add an auto-increment primary key column
    private static Dataset<Row> addAutoIncrementPrimaryKey(Dataset<Student> dataset) {
        // Add a new column with a monotonically increasing id
        Dataset<Row> indexedDataset = dataset.withColumn("id",
                functions.monotonicallyIncreasingId());

        return indexedDataset;
    }

    // Sample Student class
    public static class Student {
        private String name;

        public Student(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }
    }
}


In this example, we create a Dataset<Student> with sample data. We then define a custom function addAutoIncrementPrimaryKey() that uses the monotonicallyIncreasingId() function from org.apache.spark.sql.functions to generate unique IDs for each row. This function adds a new column "id" with auto-incrementing values to the dataset.

By using the monotonicallyIncreasingId() function, you can simulate auto-increment functionality in Spark for generating unique primary key values in a dataset.
