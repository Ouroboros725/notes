https://stackoverflow.com/questions/17660469/get-field-class-in-annotations-processor

When running your annotation processor, you don't have access to the compiled classes. The point of annotation processing is that it happens pre-compile.

Instead, you need to create an annotation processor that specifically handles your annotation type, then use the mirror API to access the field. For example:

    @SupportedAnnotationTypes("com.example.MyAnnotation")
    public class CompileTimeAnnotationProcessor extends AbstractProcessor {
    
        @Override
        public boolean process(Set<? extends TypeElement> annotations, 
                               RoundEnvironment roundEnv) {
            // Only one annotation, so just use annotations.iterator().next();
            Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(
                    annotations.iterator().next());
            Set<VariableElement> fields = ElementFilter.fieldsIn(elements);
            for (VariableElement field : fields) {
                TypeMirror fieldType = field.asType();
                String fullTypeClassName = fieldType.toString();
                // Validate fullTypeClassName
            }
            return true;
        }
    }

For the validation, you **cannot** use any classes which have yet to be compiled (including those that are about to be compiled with the annotation) using something like `MyType.class`. For these, you must use **strings only**. That is because annotation processing occurs during a pre-compiling phase known as "source generation", which is what allows you to generate source code before the compiler runs using annotations.

An example validation verifying that the field type is `java.lang.String` (which is already compiled):

    for (VariableElement field : fields) {
        TypeMirror fieldType = field.asType();
        String fullTypeClassName = fieldType.toString();
        if (!String.class.getName().equals(fullTypeClassName)) {
            processingEnv.getMessager().printMessage(
                    Kind.ERROR, "Field type must be java.lang.String", field);
        }
    }

**Resources**

* [Main APT Page][1]
* [Mirror API Javadocs (Java 7 and older)][2]
* **Edit:** [Mirror API Javadocs (Java 8)][3]
    * Note that the mirror API is now standardized in Java 8 under `javax.lang.model` and the old API is deprecated. See [this blog post][4] for more information. If you've been using the `javax` classes, then you don't need to worry.

**Edit:**

> I want to get the field type to get annotations on that type. But this does not seem like it will be possible?

Indeed it is possible! This can be done using more methods on the `TypeMirror`:

    if (fieldType.getKind() != TypeKind.DECLARED) {
        processingEnv.getMessager().printMessage(
                Kind.ERROR, "Field cannot be a generic type.", field);
    }
    DeclaredType declaredFieldType = (DeclaredType) fieldType;
    TypeElement fieldTypeElement = (TypeElement) declaredFieldType.asElement();

From here, you have two choices:

1. If the annotation you're trying to find is already compiled (i.e. it's from another library) then you can reference the class directly to get the annotation.
2. If the annotation you're trying to find is not compiled (i.e. it's being compiled in the current call to `javac` that's running the APT) then you can reference it via `AnnotationMirror` instances.

**Already Compiled**

    DifferentAnnotation diffAnn = fieldTypeElement.getAnnotation(
            DifferentAnnotation.class);
    // Process diffAnn

Very straight-forward, this gives you direct access to the annotation itself.

**Not Compiled**

Note that this solution will work regardless of whether or not the annotation is compiled, it's just not as clean as the code above.

Here are a couple methods I wrote once to extract a certain value from an annotation mirror by its class name:

    private static <T> T findAnnotationValue(Element element, String annotationClass,
            String valueName, Class<T> expectedType) {
        T ret = null;
        for (AnnotationMirror annotationMirror : element.getAnnotationMirrors()) {
            DeclaredType annotationType = annotationMirror.getAnnotationType();
            TypeElement annotationElement = (TypeElement) annotationType
                    .asElement();
            if (annotationElement.getQualifiedName().contentEquals(
                    annotationClass)) {
                ret = extractValue(annotationMirror, valueName, expectedType);
                break;
            }
        }
        return ret;
    }

    private static <T> T extractValue(AnnotationMirror annotationMirror,
            String valueName, Class<T> expectedType) {
        Map<ExecutableElement, AnnotationValue> elementValues = new HashMap<ExecutableElement, AnnotationValue>(
                annotationMirror.getElementValues());
        for (Entry<ExecutableElement, AnnotationValue> entry : elementValues
                .entrySet()) {
            if (entry.getKey().getSimpleName().contentEquals(valueName)) {
                Object value = entry.getValue().getValue();
                return expectedType.cast(value);
            }
        }
        return null;
    }

Let's say that you're looking for the `DifferentAnnotation` annotation and your source code looks like this:

    @DifferentAnnotation(name = "My Class")
    public class MyClass {
    
        @MyAnnotation
        private String field;
    
        // ...
    }

This code will print `My Class`:

    String diffAnnotationName = findAnnotationValue(fieldTypeElement,
            "com.example.DifferentAnnotation", "name", String.class);
    System.out.println(diffAnnotationName);


[1]: http://docs.oracle.com/javase/7/docs/technotes/guides/apt/
[2]: http://docs.oracle.com/javase/7/docs/jdk/api/apt/mirror/overview-summary.html
[3]: https://docs.oracle.com/javase/8/docs/api/javax/lang/model/package-summary.html
[4]: https://blogs.oracle.com/darcy/entry/an_apt_replacement