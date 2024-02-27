Assigment 3.Validate the stock prices displayed in the https://money.rediff.com/losers/bse/daily/groupall  website

1. Read the data in the XLS file.
2. Store the data in HashMap1.
3. Read the data using Selenium WebDriver.
4. Store the data in HashMap2.
5. Compare the values stored in the two HashMaps.

*******************************************************************
**Java File :** RediffMoneyTest.java

Code Logic:

Public class RediffMoneyTest {
private WebDriver driver;

    @Before
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.get("https://money.rediff.com/losers/bse/daily/groupall");
        driver.manage().window().maximize();
    }

    @After
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }

    @Test
    public void verifyDataFromTable() throws IOException {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(2));
        WebElement table = wait.until(ExpectedConditions.presenceOfElementLocated(By.xpath("//table[@class =\"dataTable\"]")));

        List<WebElement> rows = table.findElements(By.tagName("tr"));
        Map<String, String> tableData = new TreeMap<>();
        for (int i = 1; i < rows.size(); i++) {
            List<WebElement> cells = rows.get(i).findElements(By.tagName("td"));
            String key = cells.get(0).getText();
            String value = cells.get(3).getText();
            tableData.put(key, value);
        }
        String actualOutput = null;
        for (Map.Entry<String, String> entry : tableData.entrySet()) {
            actualOutput = entry.getKey() + " " + entry.getValue();
            System.out.println(actualOutput);
        }

        String excelFilePath = "src/test/Excel/MoneyRediffStockSheet.xlsx";
        Workbook workbook = null;
        if (excelFilePath.endsWith(".xlsx")) {
            workbook = new XSSFWorkbook(new FileInputStream(excelFilePath));
        } else if (excelFilePath.endsWith(".xls")) {
            workbook = new HSSFWorkbook(new FileInputStream(excelFilePath));
        } else {
            throw new IOException("Unsupported file format. Please provide an XLS or XLSX file.");
        }
        Sheet sheet = workbook.getSheetAt(0);
        String excelSheetValue = null;
        for (int i = 0; i <= sheet.getLastRowNum(); i++) {
            Row row = sheet.getRow(i);
            if (row == null) {
                continue;
            }
            for (int j = 0; j < row.getLastCellNum(); j++) {
                Cell cell = row.getCell(j, Row.MissingCellPolicy.CREATE_NULL_AS_BLANK);
                excelSheetValue = cell.toString();
                System.out.print(excelSheetValue + " ");
            }
            System.out.println();
        }
        workbook.close();

        Assert.assertThat(actualOutput, Matchers.containsString(excelSheetValue));

    }
