https://stackoverflow.com/questions/17346679/transactional-on-postconstruct-method

Quote from legacy (closed) Spring forum:
> In the @PostConstruct (as with the afterPropertiesSet from the InitializingBean interface) there is no way to ensure that all the post processing is already done, so (indeed) there can be no Transactions. The only way to ensure that that is working is by using a TransactionTemplate.


So if you would like something in your `@PostConstruct` to be executed within transaction you have to do something like this:

    @Service("something")
    public class Something {
        
        @Autowired
        @Qualifier("transactionManager")
        protected PlatformTransactionManager txManager;

        @PostConstruct
        private void init(){
            TransactionTemplate tmpl = new TransactionTemplate(txManager);
            tmpl.execute(new TransactionCallbackWithoutResult() {
                @Override
                protected void doInTransactionWithoutResult(TransactionStatus status) {
                    //PUT YOUR CALL TO SERVICE HERE
                }
            });
       }
    }
