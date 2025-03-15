# Occurify.TimeZones

Time zone and cron expression support for Occurify: Filter, manipulate, and schedule instants and periods across time zones.

## 📖 Table of Contents  
- [Overview](#Overview)
- [Installation](#installation)
- [Usage](#usage)
- [Potential Use Cases](#potential-use-cases)
    - [Use of Cron Expressions](#use-of-cron-expressions)
    - [Defining Instants](#defining-instants)
    - [Use Crons to Create Periods](#use-crons-to-create-periods)
    - [Searching Dates](#searching-dates)
- [License](#license)

## Overview

- Supports time zone based instants and periods (e.g. time of day, day, week, etc).
- Supports both forwards and backwards iteration through Cron instants and periods.
- Uses the *Cronos* library to enable Cron functionality that:
- Supports standard Cron format with optional seconds.
- Supports time zones, and performs all the date/time conversions for you.
- Does not skip occurrences, when the clock jumps forward to daylight saving time (known as Summer time).
- Does not skip interval-based occurrences, when the clock jumps backward from Summer time.
- Does not retry non-interval based occurrences, when the clock jumps backward from Summer time.

For details on the Occurify ecosystem including examples for this library, please check the following documentation: [Occurify main README](https://github.com/Occurify/Occurify).

## Installation

Occurify.TimeZones is distributed as a [NuGet package](https://www.nuget.org/packages/Occurify.TimeZones), you can install it from the official NuGet Gallery. Please use the following command to install it using the NuGet Package Manager Console window.
```
PM> Install-Package Occurify.TimeZones
```

## Usage

Rather than working with concrete instants and periods in time, Occurify allows for conceptual representation of time using instant and period timelines.

For example, rather than listing all workdays of a year to work with, you can define the concept of "all workdays", apply transformations or filters, and extract the relevant periods as needed.

The following example demonstrates how to define a period timeline, `workingHours` that includes all periods between **8 AM and 6 PM**. By subtracting weekends, we obtain a new period timeline, `workingTime`, that represents all workdays within that range:
```cs
IPeriodTimeline workingHours = TimeZonePeriods.Between(startHour: 8, endHour: 18);
IPeriodTimeline weekends = TimeZonePeriods.Days(DayOfWeek.Saturday, DayOfWeek.Sunday);
IPeriodTimeline workingTime = workingHours - weekends;
```
Now, `workingTime` represents all workdays from **8 AM and 6 PM**.

```cs
Console.WriteLine(workingTime.IsNow() ? "You should still be working" : "You can go home!");
```
Or
```cs
Console.WriteLine($"The year 1025 contained {workingTime.EnumeratePeriod(TimeZonePeriods.Year(1025)).Count()} workdays.");
Console.WriteLine($"The year 3025 will contain {workingTime.EnumeratePeriod(TimeZonePeriods.Year(3025)).Count()} workdays.");
```
The period timeline **only resolves the necessary periods when enumerated**, ensuring efficiency.

This approach allows developers to focus on what they need—such as "workdays"—without manually managing time calculations. As a result, coding becomes more intuitive and use case-driven.

## Potential Use Cases

This section presents various use cases that demonstrate Occurify’s capabilities and provide a clearer understanding of its functionality.

>**Note: Instead of using `var`, variable types are explicitly defined in the examples for improved clarity.**

### Use of Cron Expressions

```cs
ITimeline cronTimeline = TimeZoneInstants.FromCron("0 * * * *");

DateTime? previousOccurrence = cronTimeline.GetPreviousUtcInstant(DateTime.UtcNow);
DateTime? nextOccurrence = cronTimeline.GetNextUtcInstant(DateTime.UtcNow);
```

### Defining Instants

```cs
 ITimeline cronTimeline = TimeZoneInstants.DailyAt(hour: 13, minute: 45);

 DateTime? previousOccurrence = cronTimeline.GetPreviousUtcInstant(DateTime.UtcNow);
 DateTime? nextOccurrence = cronTimeline.GetNextUtcInstant(DateTime.UtcNow);
```

### Use Crons to Create Periods

The following example demonstrates a more complex example with the use of cron expressions. Not only does it allow you to create a timeline with instants, but it can also convert a cron expression directly into a period-based timeline (e.g., hours, days).

This example calculates how many working days there are if we exclude public holidays.
```cs
string[] holidayCrons = [
    "0 0 0 1 1 ?", // New Year Day
    "0 0 0 ? 5 MON#4", //Memorial Day
    "0 0 0 4 7 ?", //Independence Day
    "0 0 0 ? 9 MON#1", //Labor Day
    "0 0 0 ? 11 THU#4", //Thanksgiving
    "0 0 0 25 12 ?" //Christmas
];

IPeriodTimeline holidays = TimeZonePeriods.Days(holidayCrons.Select(TimeZoneInstants.FromCron).Combine());
IPeriodTimeline workingDays = TimeZonePeriods.Workdays();
IPeriodTimeline workingDaysWithoutHolidays = workingDays - holidays;

Console.WriteLine("Work days this year:");
foreach (Period period in workingDaysWithoutHolidays.EnumeratePeriod(TimeZonePeriods.CurrentYear()))
{
    Console.WriteLine(period.ToString(TimeZoneInfo.Local));
}
```

#### Using ReactiveX for Scheduling

Now we can use `ToBooleanObservableIncludingCurrent` from `Occurify.Reactive` to integrate with `ReactiveX` for event-driven scheduling:
```cs
workingDaysWithoutHolidays.ToBooleanObservable(scheduler).Subscribe(inPeriod =>
{
    if (inPeriod)
    {
        Console.WriteLine("Today is a working day");
        return;
    }
    Console.WriteLine("Today is NOT a working day");
});
```

### Working with Different Periods

Occurify allows us to define periods however we want. In this example we use `TimeZoneInstants.StartOfMonth(10)` to get a timeline with the start of every October. Using `AsConsecutivePeriodTimeline` turns those instants into consecutive periods that we can use to represent fiscal years.

```cs
IPeriodTimeline calendarYears = TimeZonePeriods.Years();
IPeriodTimeline fiscalYears = TimeZoneInstants.StartOfMonth(10).AsConsecutivePeriodTimeline();

Period currentCalendarYear = calendarYears.SampleAt(DateTime.UtcNow).Period!;
Period currentFiscalYear = fiscalYears.SampleAt(DateTime.UtcNow).Period!;
```

Next, we can use `EnumerateRange` to count how many workdays have passed in both the calendar and fiscal years:

```cs
IPeriodTimeline workdays = TimeZonePeriods.Workdays();
int amountOfCalendarDaysWorked = workdays.EnumerateRange(currentCalendarYear.Start!.Value, DateTime.UtcNow).Count();
int amountOfFiscalYearDaysWorked = workdays.EnumerateRange(currentFiscalYear.Start!.Value, DateTime.UtcNow).Count();
```

### Searching Dates

Here's an example of how you can find out how many Fridays there were in February of the years 1200 and 1201:

```cs
IPeriodTimeline fridaysOfFebruary = TimeZonePeriods.Months(2, TimeZoneInfo.Utc) & TimeZonePeriods.Days(DayOfWeek.Friday, TimeZoneInfo.Utc);
Period twoYears = TimeZoneInstants.StartOfYear(1200, TimeZoneInfo.Utc).To(TimeZoneInstants.EndOfYear(1201, TimeZoneInfo.Utc));

Console.WriteLine("The years 1200 and 1201 have the following fridays in february:");
foreach (var date in fridaysOfFebruary.EnumeratePeriod(twoYears))
{
    Console.WriteLine(date.Start!.Value.ToShortDateString());
}
```

## License

Copyright © 2025 Jasper Lammers. Occurify.TimeZones is licensed under [The MIT License (MIT)](https://github.com/Occurify/Occurify.TimeZones?tab=MIT-1-ov-file).