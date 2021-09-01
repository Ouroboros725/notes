https://stackoverflow.com/questions/16570073/whats-the-jackson-deserialization-equivalent-of-jsonunwrapped

You can use `@JsonCreator` with `@JsonProperty` for each field:

	@JsonCreator
	public Parent(@JsonProperty("age") Integer age, @JsonProperty("firstName") String firstName,
            @JsonProperty("lastName") String lastName) {
		this.age = age;
		this.name = new Name(firstName, lastName);
	}

Jackson does type checking and unknown field checking for you in this case.