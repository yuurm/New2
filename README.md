Для реализации указанных требований в Spring Boot приложении с использованием конфигурационных карт (ConfigMap) в OpenShift, можно следовать следующему плану:

### Шаги реализации

#### 1. Создание конфигурационного блока для путей и типов ТК

Создайте ConfigMap в OpenShift для хранения соответствий между путями и типами технологий. 

##### Пример ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: path-to-tk-type-config
data:
  path-to-tk-type: |
    package/bh=Толстый клиент
    package/configmaps=Конфиг
```

##### Примените ConfigMap:

```bash
oc apply -f path-to-tk-type-config.yml
```

#### 2. Обновление Spring Boot приложения для чтения данных из ConfigMap

##### 2.1. Добавьте зависимости в `pom.xml`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-config</artifactId>
</dependency>
```

##### 2.2. Настройка приложения для чтения ConfigMap

Создайте файл `bootstrap.yml` в `src/main/resources`:

```yaml
spring:
  cloud:
    kubernetes:
      config:
        name: path-to-tk-type-config
        namespace: <your-namespace> # Замените на ваш namespace в OpenShift
        enabled: true
```

##### 2.3. Создайте класс для чтения и хранения конфигурации

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import java.util.HashMap;
import java.util.Map;

@Component
public class PathToTkTypeConfig {

    @Value("${path-to-tk-type}")
    private String pathToTkTypeString;

    private Map<String, String> pathToTkTypeMap = new HashMap<>();

    @PostConstruct
    public void init() {
        String[] mappings = pathToTkTypeString.split("\n");
        for (String mapping : mappings) {
            String[] parts = mapping.split("=");
            if (parts.length == 2) {
                pathToTkTypeMap.put(parts[0].trim(), parts[1].trim());
            }
        }
    }

    public String getTkTypeForPath(String path) {
        return pathToTkTypeMap.get(path);
    }
}
```

#### 3. Разбор SBOM и анализ блока `files`

##### 3.1. Создайте сервис для разбора SBOM и анализа файлов

```java
import org.springframework.stereotype.Service;
import java.util.List;
import java.util.Optional;

@Service
public class SbomAnalyzerService {

    private final PathToTkTypeConfig pathToTkTypeConfig;
    private final TechnologyRepository technologyRepository;

    public SbomAnalyzerService(PathToTkTypeConfig pathToTkTypeConfig, TechnologyRepository technologyRepository) {
        this.pathToTkTypeConfig = pathToTkTypeConfig;
        this.technologyRepository = technologyRepository;
    }

    public void analyzeSbom(Sbom sbom) {
        List<File> files = sbom.getFiles();
        for (File file : files) {
            String filePath = file.getPath();
            Optional<String> optionalTkType = getTkTypeFromPath(filePath);
            if (optionalTkType.isPresent()) {
                String tkType = optionalTkType.get();
                saveOrUpdateTechnologyComponent(tkType, file);
            }
        }
    }

    private Optional<String> getTkTypeFromPath(String path) {
        return Optional.ofNullable(pathToTkTypeConfig.getTkTypeForPath(path));
    }

    private void saveOrUpdateTechnologyComponent(String tkType, File file) {
        TechnologyComponent existingComponent = technologyRepository.findByName(tkType);
        if (existingComponent == null) {
            TechnologyComponent newComponent = new TechnologyComponent();
            newComponent.setName(tkType);
            newComponent.setVersion(file.getVersion());
            newComponent.setSource("SBOM");
            technologyRepository.save(newComponent);
        } else {
            existingComponent.setVersion(file.getVersion());
            technologyRepository.save(existingComponent);
        }
    }
}
```

##### 3.2. Добавьте модели для SBOM и TechnologyComponent

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class TechnologyComponent {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String version;
    private String source;

    // getters and setters
}

public class Sbom {
    private List<File> files;

    // getters and setters

    public static class File {
        private String path;
        private String version;

        // getters and setters
    }
}
```

##### 3.3. Создайте репозиторий для TechnologyComponent

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface TechnologyRepository extends JpaRepository<TechnologyComponent, Long> {
    TechnologyComponent findByName(String name);
}
```

#### 4. Обновите контроллер для обработки SBOM

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SbomController {

    private final SbomAnalyzerService sbomAnalyzerService;

    public SbomController(SbomAnalyzerService sbomAnalyzerService) {
        this.sbomAnalyzerService = sbomAnalyzerService;
    }

    @PostMapping("/api/sbom/analyze")
    public void analyzeSbom(@RequestBody Sbom sbom) {
        sbomAnalyzerService.analyzeSbom(sbom);
    }
}
```

Таким образом, вы сможете создать универсальный механизм для разбора SBOM, анализа файлов и сопоставления типов технологий на основе конфигурации, хранящейся в OpenShift ConfigMap.
