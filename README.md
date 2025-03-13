# Occurify.TimeZones

Time zone and cron expression support for Occurify: Filter, manipulate, and schedule instants and periods across time zones.

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

Rather than working with concrete instants and periods in time, Occurify allows for conceptual representation of time using intstant and period timelines.

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

## License

Copyright © 2025 Jasper Lammers. Occurify.TimeZones is licensed under The MIT License (MIT).