---
title: 'Get All Unique Timezone In Rails'
date: 2021-03-18T01:13:03+06:00
draft: false
---

### Rails Timezone

There is only 35 Unique TimeZone worldwide ( According to Rails Timezone Class ). We can get a list of all the unique time happening worldwide by running the following snippet from rails console and see it ourselves. :smile:

```ruby

 ActiveSupport::TimeZone.all.map(&:name).map do |zone|
   Time.now.in_time_zone(zone).strftime('%F %T %:z') }.uniq
 end

```

To get the full list of 150 timezones

```ruby

 ActiveSupport::TimeZone.all.map(&:name).map do |zone|
  zone + ' : ' + Time.now.in_time_zone(zone).strftime('%F %T %:z') }
 end

```

Thank you for reading!
