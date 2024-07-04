@Autowired
    private TransactionRepository transactionRepository;

    @Autowired
    private CreditCardRepository creditCardRepository;

    @Autowired
    private MerchantRepository merchantRepository;

    @Override
    public Transaction postTransaction(Transaction transaction) {
        // Validate credit card number
        CreditCard creditCard = creditCardRepository.getByNumber(transaction.getCreditcard().getCreditcardNumber());
        if (creditCard == null) {
            throw new IllegalArgumentException("Invalid credit card number");
        }
        if(!creditCard.getCreditcardstatus().equals("open")) {
        	 throw new IllegalArgumentException("credit card inactive");
        }
        // Validate merchant code
        Merchant merchant = merchantRepository.findById(transaction.getMerchant().getMerchantID())
            .orElseThrow(() -> new IllegalArgumentException("Invalid merchant code"));
        if(!(merchant.getMerchantStatus()==MerchantStatus.ACTIVE)) {
       	 throw new IllegalArgumentException("merchant is not available");
       }

        // Validate available credit limit
        if (creditCard.getCreditLimit() < transaction.getTransactionAmount()) {
            throw new IllegalArgumentException("Insufficient credit limit");
        }

        // Set other necessary transaction details
        transaction.setCreditcard(creditCard);
        transaction.setMerchant(merchant);
        transaction.setAuthCode(generateAuthCode()); // Assuming this method generates a unique 6-digit auth code

        return transactionRepository.save(transaction);
    }

    private String generateAuthCode() {
        return String.format("%06d", (int)(Math.random() * 1_000_000));
    }
