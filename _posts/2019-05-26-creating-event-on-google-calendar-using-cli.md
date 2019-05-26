---
title: Creating Event on Google Calendar using CLI
layout: post
date: '2019-05-26'
category: ruby
---

If you regularly use google calendar to create event then you will find the terminal to be of much help. 
We can write a script in ruby that we will make executable from terminal which will create an event on google calendar. 

`reminder 2019-06-01 2:13PM 'text description for the event'`

We will use `google-api-client` for integrating the app with google calendar. 

We need to create google calendar service object through which the event is sent to the server.

```ruby
 def service
    service = Google::Apis::CalendarV3::CalendarService.new
    service.client_options.application_name = "App Name"
    service.authorization = authorize
    service
  end

  def authorize
    authorizer = Google::Auth::ServiceAccountCredentials.make_creds(
      json_key_io: File.open((ENV['GCP_CONFIG_PATH']).to_s),
      scope: Google::Apis::CalendarV3::AUTH_CALENDAR
    )
    authorizer.fetch_access_token!
    authorizer
  end
```

The `add_event_to_calendar` method is called by passing the `service` object created in the above method and the event hash which includes the start-time, end-time and summary for the event. 

```ruby
 def add_event_to_calendar(service, event)
    begin
      new_event = Google::Apis::CalendarV3::Event.new(
        summary: event[:summary],
        start: { date_time: event[:start_time].rfc3339 },
        end: { date_time: event[:end_time].rfc3339 }
        )
      res = service.insert_event("your-email@gmail.com", new_event)
    rescue Google::Apis::ClientError => e
      puts "Check if your email is correct or not"
    else
      if res.status == "confirmed"
        ap "Event has been registered successfully on calendar #{res.organizer.display_name}"
      else
        puts "Unknown Error"
      end
    end
  end
```

This is a simple script which can be extended to list all event, and delete the event too. If you want to take a look at full code, please find the link here.  [ reminder.rb](https://github.com/aadeshere1/reminder/blob/master/reminder.rb)

Once the script is completed, then we can create the symlink to run the code from terminal. For that we have to make the script executable. Add `#!/usr/bin/env ruby` on top of our code to let the terminal know that we want to execute the script as ruby code.

```ruby
#!/usr/bin/env ruby
... # removed code for brevity
```

We can make it executable by `chmod 755 reminder` and create a symlink `ln -s $PWD/reminder /usr/local/bin
`
	P.S. I wrote the code to teach one of my friend who was interested to learn ruby. So, there are lot of places where the code can be made much better.