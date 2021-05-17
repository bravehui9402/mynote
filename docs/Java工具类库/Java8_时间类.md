```java
      //获取今天的日期
      LocalDate today = LocalDate.now(); //2021-05-17
	  
	  //获取年、月、日信息
	  LocalDate today = LocalDate.now(); //2021-05-17
      int year = today.getYear(); //2021
      int month = today.getMonthValue();//5
      int day = today.getDayOfMonth();//17
      
	  //处理特定日期
	  LocalDate dateOfBirth = LocalDate.of(2021, 05, 17);

	  //判断两个日期是否相等
	  LocalDate date1 = LocalDate.of(2021, 05, 14);
	  LocalDate today = LocalDate.now();
	  boolean result= date1.equals(today);
		
	  //在Java 8中检查像生日这种周期性事件
	  LocalDate today = LocalDate.now();
      LocalDate dateOfBirth = LocalDate.of(2010, 01, 14);
      MonthDay birthday = MonthDay.of(dateOfBirth.getMonth(), dateOfBirth.getDayOfMonth());
      MonthDay currentMonthDay = MonthDay.from(today);
	  boolean result =  currentMonthDay.equals(birthday)
          
      //计算一周后的日期
	  LocalDate nextWeek = today.plus(1, ChronoUnit.WEEKS);    
      
	  //计算一年前或一年后的日期
	  LocalDate previousYear = today.minus(1, ChronoUnit.YEARS);//一年前
	  LocalDate nextYear = today.plus(1, YEARS);//一年后

      //用Java判断日期是早于还是晚于另一个日期
       LocalDate tomorrow = LocalDate.of(2014, 1, 15);
       if(tommorow.isAfter(today)){
           System.out.println("Tomorrow comes after today");
       }
       LocalDate yesterday = today.minus(1, DAYS);
       if(yesterday.isBefore(today)){
           System.out.println("Yesterday is day before today");}

	   //在Java 8中处理时区
        oneId america = ZoneId.of("America/New_York");
        LocalDateTime localtDateAndTime = LocalDateTime.now();
        ZonedDateTime dateAndTimeInNewYork  = ZonedDateTime.of(localtDateAndTime, america );


		//在Java 8中检查闰年
	    LocalDate today = LocalDate.now();
		today.isLeapYear()
            
        //计算两个日期之间的天数和月数
        LocalDate java8Release = LocalDate.of(2014, Month.MARCH, 14);
		Period periodToNextJavaRelease = Period.between(today, java8Release);  
	    periodToNextJavaRelease.getMonths();//月数
	    periodToNextJavaRelease.getDays() ; //天数
 		periodToNextJavaRelease.getYears(); //年数
-----------------------------------------------------------------------------------------------------
	  //获取当前时间
	  LocalTime time = LocalTime.now();	//19:49:18.235

	  //在现有的时间上增加小时
	  LocalTime time = LocalTime.now();
      LocalTime newTime = time.plusHours(2); // 在当前时间增加小时、分、秒
	  LocalTime localTime = time.withHour(10);// 在当前时间减少时、分、秒

	  //获取当前的时间戳
	  Instant timestamp = Instant.now()
------------------------------------------------------------------------------------------------------
      //在Java 8中使用预定义的格式化工具去解析或格式化日期
	  String dayAfterTommorrow = "20140116";
	  LocalDate formatted = LocalDate.parse(dayAfterTommorrow,DateTimeFormatter.BASIC_ISO_DATE);

      String goodFriday = "Apr 18 2014";
      try {
          DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MMM dd yyyy");
          LocalDate holiday = LocalDate.parse(goodFriday, formatter);
          System.out.printf("Successfully parsed String %s, date is %s%n", goodFriday, holiday);
      } catch (DateTimeParseException ex) {
          System.out.printf("%s is not parsable!%n", goodFriday);
          ex.printStackTrace();
      }


        //日期转换成字符串
	    LocalDateTime arrivalDate  = LocalDateTime.now();
        try {
            DateTimeFormatter format = DateTimeFormatter.ofPattern("MMM dd yyyy  hh:mm a");
            String landing = arrivalDate.format(format);
            System.out.printf("Arriving at :  %s %n", landing);
        } catch (DateTimeException ex) {
            System.out.printf("%s can't be formatted!%n", arrivalDate);
            ex.printStackTrace();
        }
```

