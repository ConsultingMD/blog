---
layout: post
title: "PagerDuty and Confluence"
date: 2018-01-12 18:19:49 +0000
comments: true
author: "Bryan Kroger bryan.kroger@grandrounds.com"
categories: automation
---
My first project here at Grand Rounds was to pick up one of my favorite topics of IT and operations 
in general: on-call rotations!  Making sure we have accurate alarming and alerting never seems like the
most glamorous work in the world, but it is often very important and highly impactful.

To that end, as with most things, we want to start with a data-driven approach to identifying and resolving
our problems.  This is a pattern that I've used several times over the years for a fun little hack which
allows us to build reports based on PagerDuty incident data.

This gives us two graphs:

* Number of incidents per service.
* Amount of time each service spends in incident ( on average ).

We fully expect that this code is a little rough around the edges, and that's okay.  We're building things
here that are going to give us ideas and insight into our bigger picture items.  Ideally we want to
spend as little time as possible gold plating this code because we're not 100% sure this will be something
that we intend to live forever.  Just like in the startup business world we want to create the Minimal
Viable Product ( MVP ) which allows us to prove out a business case.

In this case we will prove out the value of incident reporting data.  This is a perfect example of a product
that is valuable today, but becomes less valuable over time because it allows us to refine a process.

We start this with a high-level HTTPClient.  Our nickname for this is called Groot: 
Grand Rounds Operational and Organizational Tool.  We're all fans of the comic books over here, 
so we often name our projects after Marvel characters.  I haven't seen anything from the DC 
realm quite yet, but maybe someday.

```ruby
class GrootHTTPClient
  @url
  @headers

  attr_accessor :url, :headers
  def initialize
    @url = get_url
    @headers = get_headers
  end

  def get_headers
    {
        'Content-Type' => 'application/json'
    }
  end
end
```

This is a very simple little base class to get us started.  Now let's build a specific client on top of this:

```ruby
class ConfluenceClient < GrootHTTPClient
  @username
  @password

  attr_accessor :username, :password
  def initialize
    @username = ENV['JIRA_USERNAME'] ||= DEFAULT_USERNAME
    @password = ENV['JIRA_PASSWORD'] ||= File.read(format('%s/.wiki_pass', ENV['HOME'])).chomp
    super
  end

  def get_url
    'https://HOSTED_SITE.atlassian.net/wiki'
  end

  def get_headers
    super.merge({
      Authorization: format('Basic %s', Base64.encode64(format('%s:%s', @username, @password)))
    })
  end

  def get_page_by_id(page_id)
    uri = format('%s/rest/api/content/%i?status=any', @url, page_id )
    Log.debug(format('URI: %s', uri))
    res = RestClient.get(uri, @headers)
    JSON.parse res.body
  end

  def get_page(space_key, title)
    params = { spaceKey: space_key, title: title, expand: 'version,body.view,space' }
    param_str = params.map { |k,v| format('%s=%s', k, URI.encode(v)) }.join('&')
    uri = format('%s/rest/api/content?%s', @url, param_str)
    Log.debug(format('URI: %s', uri))
    res = RestClient.get(uri, @headers)
    JSON.parse( res.body )['results'].first
  end

  def update_page(page, new_content)
    page_id = page['id'].to_i
    current_version = page['version']['number']

    params = { space: { key: page['space']['key'] }, title: page['title'], type: 'page' }
    current_version += 1
    params[:version] = {
        hidden: false,
        number: current_version,
        minorEdit: true
    }

    params[:body] = {
        storage: {
            value: new_content,
            representation: 'storage'
        }
    }

    uri = format('%s/rest/api/content/%i', @url, page_id)
    Log.debug(format('URI: %s', uri))
    headers = @headers
    headers['Accept'] = 'application/json'
    headers['Content-Type'] = 'application/json'
    RestClient.put(uri, params.to_json, headers)
  end
end
```

Now let's build a quick and dirty client for PagerDuty:

```ruby
class PDClient
  @key

  attr_accessor :key
  def initialize( key )
    @key = key
  end

  def sec_to_human( total_seconds )
    seconds = total_seconds % 60
    minutes = (total_seconds / 60) % 60
    hours = total_seconds / (60 * 60)

    format("%02d:%02d:%02d", hours, minutes, seconds)
  end

  def headers
    {
      Accept: 'application/vnd.pagerduty+json;version=2',
      Authorization: format('Token token=%s', @key)
    }
  end

  def uri
    'https://api.pagerduty.com'
  end

  def schedules
    url = format('%s/schedules', uri)
    RestClient.get( url, headers )
  end

  def services
    url = format('%s/services', uri)
    RestClient.get( url, headers )
  end

  def service( id )
    url = format('%s/service/%i', uri, id)
    RestClient.get( url, headers )
  end

  def service_events( id )
    since = '2017-12-01T00:00:00'
    to = '2017-12-31T23:59:59'
    url = format('%s/incidents?since=%s&until=%s&service_ids[]=%s', uri, since, to, id)
    RestClient.get( url, headers )
  end
end
```

Pretty simple code here.  Basically we're going to use this to get a current page, and then update the page later on.  Now lets tie this all together with a Rake task:

```ruby
desc 'Create a report on incidents based on services.'
task :report_services do |t,args|
  report = {}
  pd_key = File.read(format('%s/.pager_duty/api_key', ENV['HOME']))
  pdc = PDClient.new( pd_key )
  pdc.services['services'].each do |service|
    service_events = pdc.service_events( service['id'] )

    report[service['name']] = { cnt: service_events['incidents'].size, total_time: 0.0, avg_time: 0.0 }

    service_events['incidents'].each do |i|
      created_at = Time.parse(i['created_at'])
      resolved_at = Time.parse(i['last_status_change_at'])
      ticket_time = resolved_at.to_i - created_at.to_i

      report[service['name']][:total_time] += ticket_time
    end

    if report[service['name']][:cnt] > 0
      report[service['name']][:avg_time] = (report[service['name']][:total_time] / report[service['name']][:cnt])
    end
  end

  report_by_incidents = report.sort{|a,b| b[1][:cnt] <=> a[1][:cnt] }
  report_by_ticket_time = report.sort{|a,b| b[1][:avg_time] <=> a[1][:avg_time] }

  jira_client = JiraClient.new
  jira_page = jira_client.get_page('~bryan.kroger', 'PagerDuty report 2017-12' )

  page = ''
  page << '<ac:structured-macro ac:macro-id="c3c570b3-e820-4787-9242-05af79a10c55" ac:name="chart" ac:schema-version="1">'
  page << '<ac:parameter ac:name="3D">true</ac:parameter>'
  page << '<ac:parameter ac:name="width">500</ac:parameter>'
  page << '<ac:parameter ac:name="title">PagerDuty alerts by service.</ac:parameter>'
  page << '<ac:parameter ac:name="legend">false</ac:parameter>'
  page << '<ac:parameter ac:name="type">bar</ac:parameter>'
  page << '<ac:parameter ac:name="height">350</ac:parameter>'
  page << '<ac:parameter ac:name="dataDisplay">after</ac:parameter>'
  page << '<ac:rich-text-body>'
  page << '<table> <tbody> <tr> <th> <p>Service name</p> </th> <th> <p>Count</p> </th> </tr>'
  report_by_incidents.each do |name, info|
    page << format('<tr><td>%s</td><td>%s</td></tr>', name, info[:cnt])
  end
  page << '</tbody></table>'
  page << '</ac:rich-text-body>'
  page << '</ac:structured-macro>'

  page << '<ac:structured-macro ac:macro-id="c3c570b3-e820-4787-9242-05af79a10c55" ac:name="chart" ac:schema-version="1">'
  page << '<ac:parameter ac:name="3D">false</ac:parameter>'
  page << '<ac:parameter ac:name="width">500</ac:parameter>'
  page << '<ac:parameter ac:name="title">Average resolution time by service.</ac:parameter>'
  page << '<ac:parameter ac:name="legend">false</ac:parameter>'
  page << '<ac:parameter ac:name="type">bar</ac:parameter>'
  page << '<ac:parameter ac:name="height">350</ac:parameter>'
  page << '<ac:parameter ac:name="dataDisplay">after</ac:parameter>'
  page << '<ac:rich-text-body>'
  page << '<table> <tbody> <tr> <th> <p>Service name</p> </th> <th> <p>Average resolution time</p> </th> </tr>'
  report_by_ticket_time.each do |name, info|
    page << format('<tr><td>%s</td><td>%s</td></tr>', name, pdc.sec_to_human(info[:avg_time]))
  end
  page << '</tbody></table>'
  page << '</ac:rich-text-body>'
  page << '</ac:structured-macro>'

  jira_client.update_page( jira_page, page )
end
```

And there we have it.  A data-driven approach to incident management.  Now we can use this data
to drive value in our operations engagements and activities.

