import com.github.javaparser.StaticJavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.body.ClassOrInterfaceDeclaration;
import com.github.javaparser.ast.body.MethodDeclaration;
import com.github.javaparser.ast.body.VariableDeclarator;
import com.github.javaparser.ast.expr.StringLiteralExpr;
import com.github.javaparser.ast.visitor.VoidVisitorAdapter;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.HashSet;
import java.util.Set;

public class CodeAnalyzer {

    private static final String PROJECT_DIR = "path/to/your/project";

    public static void main(String[] args) throws IOException {
        Set<String> fileReferences = new HashSet<>();
        Set<String> classes = new HashSet<>();
        Set<String> usedClasses = new HashSet<>();

        Files.walk(Paths.get(PROJECT_DIR))
                .filter(Files::isRegularFile)
                .filter(path -> path.toString().endsWith(".java"))
                .forEach(path -> {
                    try {
                        CompilationUnit cu = StaticJavaParser.parse(path);
                        cu.accept(new FileReferenceCollector(), fileReferences);
                        cu.accept(new ClassCollector(), classes);
                        cu.accept(new UsedClassCollector(), usedClasses);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                });

        System.out.println("File References:");
        for (String fileReference : fileReferences) {
            if (!new File(PROJECT_DIR + File.separator + fileReference).exists()) {
                System.out.println(fileReference + " does not exist.");
            }
        }

        System.out.println("\nUnused Classes:");
        for (String cls : classes) {
            if (!usedClasses.contains(cls)) {
                System.out.println(cls);
            }
        }
    }

    private static class FileReferenceCollector extends VoidVisitorAdapter<Set<String>> {
        @Override
        public void visit(StringLiteralExpr n, Set<String> collector) {
            String value = n.getValue();
            if (value.matches(".*\\.(txt|csv|xml|json|properties)")) {
                collector.add(value);
            }
            super.visit(n, collector);
        }
    }

    private static class ClassCollector extends VoidVisitorAdapter<Set<String>> {
        @Override
        public void visit(ClassOrInterfaceDeclaration n, Set<String> collector) {
            collector.add(n.getNameAsString());
            super.visit(n, collector);
        }
    }

    private static class UsedClassCollector extends VoidVisitorAdapter<Set<String>> {
        @Override
        public void visit(MethodDeclaration n, Set<String> collector) {
            n.getBody().ifPresent(body -> body.findAll(ClassOrInterfaceDeclaration.class)
                    .forEach(cls -> collector.add(cls.getNameAsString())));
            super.visit(n, collector);
        }
    }
}
<dependencies>
    <dependency>
        <groupId>com.github.javaparser</groupId>
        <artifactId>javaparser-core</artifactId>
        <version>3.23.0</version>
    </dependency>
</dependencies>
