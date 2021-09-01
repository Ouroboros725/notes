https://stackoverflow.com/questions/28989841/how-to-map-elements-of-the-list-to-their-indices-using-java-8-streams

You could do something like this:

    public Robots(List<String> names) {
        this.list = IntStream.range(0, names.size())
                             .mapToObj(i -> new Robot(i, names.get(i)))
                             .collect(collectingAndThen(toList(), Collections::unmodifiableList));
    }

However it may not be as efficient depending on the underlying implementation of the list. You could grab an iterator from the `IntStream`; then calling `next()` in the `mapToObj`.

As an alternative, the [proton-pack][1] library defines the `zipWithIndex` functionality for streams:

     this.list = StreamUtils.zipWithIndex(names.stream())
                            .map(i -> new Robot(i.getIndex(), i.getValue()))
                            .collect(collectingAndThen(toList(), Collections::unmodifiableList));


[1]: https://github.com/poetix/protonpack