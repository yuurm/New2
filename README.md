import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class DisableMethodAspect {
    
    @Pointcut("execution(* com.example.MyClass.method1(..))")
    public void disableMethod1() {}
    
    @Pointcut("execution(* com.example.MyClass.method2(..))")
    public void disableMethod2() {}
    
    @Around("disableMethod1()")
    public Object disableMethod1(ProceedingJoinPoint joinPoint) throws Throwable {
        // Просто возвращаем null или другое значение по умолчанию
        return null; // или что-то другое в зависимости от требований
    }
    
    @Around("disableMethod2()")
    public Object disableMethod2(ProceedingJoinPoint joinPoint) throws Throwable {
        // Просто возвращаем null или другое значение по умолчанию
        return null; // или что-то другое в зависимости от требований
    }
}
