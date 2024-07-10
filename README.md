import java.io.File;
import java.util.ArrayList;
import java.util.List;

public class FindEmptyFiles {

    public static void main(String[] args) {
        String projectDir = "/path/to/your/project"; // Укажите путь к вашему проекту
        List<File> emptyFiles = findEmptyFiles(new File(projectDir));
        for (File emptyFile : emptyFiles) {
            System.out.println(emptyFile.getAbsolutePath());
        }
    }

    public static List<File> findEmptyFiles(File rootDir) {
        List<File> emptyFiles = new ArrayList<>();
        if (rootDir.isDirectory()) {
            File[] files = rootDir.listFiles();
            if (files != null) {
                for (File file : files) {
                    if (file.isFile() && file.length() == 0) {
                        emptyFiles.add(file);
                    } else if (file.isDirectory()) {
                        emptyFiles.addAll(findEmptyFiles(file));
                    }
                }
            }
        }
        return emptyFiles;
    }
}
