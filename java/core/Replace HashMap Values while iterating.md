https://stackoverflow.com/questions/10993403/how-to-replace-hashmap-values-while-iterating-over-them-in-java

**Using Java 8:**

    map.replaceAll((k, v) -> v - 20);


----------


**Using Java 7 or older:**

You can iterate over the entries and update the values as follows:

    for (Map.Entry<Key, Long> entry : playerCooldowns.entrySet()) {
        entry.setValue(entry.getValue() - 20);
    }

