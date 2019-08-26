---
layout: post
title:      "Using RestClient to fetch to Yelp Api"
date:       2019-08-26 02:24:06 +0000
permalink:  using_restclient_to_fetch_to_yelp_api
---


I promised that I would do this instructional because I was stuck for 3 days trying to figure out how to use Faraday which was not successful whatsoever! I know this will help someone out there and that is what makes this post worth it! Okay enough into why I did this lets get to work.

The objective is to get the React Frontend to send a fetch request to Rails, the backend that then sends a RestCleint::Request to Yelp which responds to the React fetch with the correct data that we wanted from Yelp. **The reason we use RestClient and NOT fetch in the React Frontend is due to the fact that we need to keep our API secret! Any time we ever need to use a personal key for authorization when sending requests we need to be doing it from the Backend to keep our App secure!** This is really important because I did not understand it for a second when I was told that I had to fetch to my Backend just to get the results when I could easily use Fetch in React. Luckily I thought about it for a minute and it clicked. This is the **ENTIRE** reason we use RestClient instead of fetch on the Frontend. Reread that bolded sentence to make sure you understand the whole point of this lesson if you do not fully.

First off to make sure we can use RestClient and JSON to parse the data we get from the RestClient::Request, add both `rest-client` and `json` to your Gemfile:

gem 'rest-client'
gem 'json'

Run `bundle install`

Perfect!

To begin: we are going to be requesting the businesses that we want with the params of Term and Location which is set up in my SearchBusinessesInput Component

```
import React, {Component} from 'react'
import { fetchBusinesses } from '../../actions/fetchBusinesses'
import { connect } from 'react-redux'
const initialState = {
    term: '',
    location: ''
}

export class SearchBusinessesInput extends Component {
    constructor(props){
        super(props)
        this.state={
            term: '',
            location: ''
        }
    }

    handleChange = (event) => {
        this.setState({
            [event.target.name]: event.target.value
        })
    }

    handleSubmit = (event) => {
        event.preventDefault()
        this.props.fetchBusinesses(this.state)
        this.setState(initialState)
    }

    render(){
        var divStyle={position:'relative',
            bottom: '-30px'}
        return(
     
            <div className="input" style={divStyle}>
                <form onSubmit={this.handleSubmit}>
                    <label>What Are You Looking For?</label>
										
										//term param goes here
										
                    <input type="text" name="term" value={this.state.term} onChange={this.handleChange}/>
                    {"\n"}
                    <label>Location:</label>
										
										//location param will be entered here
										
                    <input type="text" name="location" value={this.state.location} onChange={this.handleChange}/>
                    <input type="submit"/>
                </form>
            </div>
        )
    }
};

export default connect(null, {fetchBusinesses})(SearchBusinessesInput)
```

handleSubmit then calls the `fetchBusinesses` action which sends off our state to be passed in as props to our FetchBusinesses Action

```
// props = {term: "sushi", location: "San Francisco"} these inputs are interchangable!
export const fetchBusinesses = (props) => {
    return (dispatch) => {
		
		//using Thunk middleware to hold off on displaying data that is not ready
		
        dispatch({type: 'LOADING_BUSINESSES'})
				
				//send off fetch to route I have defined inside of my config/routes.rb which is connected to my FetchController
				
        return fetch(`http://localhost:3000/search_businesses?term=${props.term}&location=${props.location}`)
				
				// here is where we will get our returning data after we hit our Backend, we will return to this Action further on
        .then(resp => resp.json())
				
				//sending businesses in to our reducer to store them in state and do what needs to be done in this file
        .then(businesses => dispatch({type:'FETCH_BUSINESSES', payload: businesses}))
    }
}
```

Config Routes: `  get '/search_businesses', to: 'fetch#search_businesses'` This route allows me to 
connect to the right route in my FetchController

here is my `.env` file for later reference : `YELP_API_KEY=YOUR OWN YELP FUSION API KEY`


```
require 'pry'
require "json"
require 'rest-client' 
class FetchController < ApplicationController


    def search_businesses
		
		#params = {"term"=>"american", "location"=>"Houston", "controller"=>"fetch", "action"=>"search_businesses"}
		#grabing Term and Location from params
		
        term = params[:term]  
        location = params[:location]  
				
			#setting RestClient::Request equal to the variable* response,* which holds the actual response from RestClient
			#Grab my Yelp_API_Key from my .env file
			
        response = RestClient::Request.execute(
            method: "GET",
            url: "https://api.yelp.com/v3/businesses/search?term=#{term}&location=#{location}",  
            headers: { Authorization: "Bearer #{ENV["YELP_API_KEY"]}" }  
        )    
				
        results = JSON.parse(response.body)
        businesses = results['businesses']
				
				#Here I am storing the data inside of my own database for persisitency, you also need to set up a model for Business as well a migration to store these attributes :)
				
        send_businesses_to_database = businesses.each do |business|
            Business.find_or_create_by(
            name: business['name'],
            image_url: business['image_url'],
            category: business['categories'][0]['title'],
            rating: business['rating'],
            price: business['price'],
            address1: business['location']['address1'],
            address2: business['location']['address2'],
            city: business['location']['city'],
            zip_code: business['location']['zip_code'],
            country: business['location']['country'],
            state: business['location']['state']
            )
        end
				
				#then rendering the businesses I get when I requested to the Yelp API
				
        render json: businesses
    end
		
		....
		
	end
```

Now we are back in the Action of FetchBusinesses!

```
export const fetchBusinesses = (props) => {
    return (dispatch) => {
        dispatch({type: 'LOADING_BUSINESSES'})
        return fetch(`http://localhost:3000/search_businesses?term=${props.term}&location=${props.location}`)
				
				// we are here now with the data that we got from our Rails Backend! Woo hoo!
				
        .then(resp => resp.json())
        .then(businesses => dispatch({type:'FETCH_BUSINESSES', payload: businesses}))
    }
}
```

And that is how you Send a request from your React Frontend to your Rails Backend, grab the data from Yelp and return it to then store it in your global state in your Reducer! I will post the rest of my `config/routes.rb` and my `FetchController` to leave you with how to grab the Business' Reviews and to search for Events in your area. The input Components should be set up realtively identical to how I did my `SearchBusinessesInput.js` file. Also the FetchReviews and FetchEvents Actions should be set up identically, of course the urls should be different as well as the paylod variable for those actions. 

Rest of my FetchController 

```
require 'pry'
require "json"
require 'rest-client' 
class FetchController < ApplicationController


    def search_businesses
        term = params[:term]  
        location = params[:location]  
        response = RestClient::Request.execute(
            method: "GET",
            url: "https://api.yelp.com/v3/businesses/search?term=#{term}&location=#{location}",  
            headers: { Authorization: "Bearer #{ENV["YELP_API_KEY"]}" }  
        )    
        results = JSON.parse(response.body)
        businesses = results['businesses']
        send_businesses_to_database = businesses.each do |business|
            Business.find_or_create_by(
            name: business['name'],
            image_url: business['image_url'],
            category: business['categories'][0]['title'],
            rating: business['rating'],
            price: business['price'],
            address1: business['location']['address1'],
            address2: business['location']['address2'],
            city: business['location']['city'],
            zip_code: business['location']['zip_code'],
            country: business['location']['country'],
            state: business['location']['state']
            )
        end
        render json: businesses
    end

    def search_events
        location = params[:location]
        response = RestClient::Request.execute(
            method: "GET",
            url: "https://api.yelp.com/v3/events?location=#{location}",  
            headers: { Authorization: "Bearer #{ENV["YELP_API_KEY"]}" }  
        )    
        results = JSON.parse(response.body)
        events = results['events']
        send_events_to_database = events.each do |event|
            Event.find_or_create_by(
            attending_count: event['attending_count'],
            category: event['category'],
            cost: event['cost'],
            description: event['description'],
            image_url: event['image_url'],
            interested_count: event['interested_count'],
            is_canceled: event['is_canceled'],
            is_free: event['is_free'],
            name: event['name'],
            time_end: event['time_end'],
            time_start: event['time_start'],
            address1: event['location']['address1'],
            address2: event['location']['address2'],
            city: event['location']['city'],
            zip_code: event['location']['zip_code'],
            country: event['location']['country'],
            state: event['location']['state']
            )
        end
        render json: events
    end

    
    def business_reviews
    id = params[:id]
    response = RestClient::Request.execute(
        method: 'GET',
        url: "https://api.yelp.com/v3/businesses/#{id}/reviews",
        headers: {Authorization: "Bearer #{ENV["YELP_API_KEY"]}"}
    )
    results = JSON.parse(response)
    reviews = results['reviews']
    send_reviews_to_database = reviews.each do |review|
        Review.find_or_create_by(
        text: review['text'],
        rating: review['rating'],
        user_name: review['user']['name']
        )
    end
    render json: reviews
    end

end
```

Config/routes.rb

```
Rails.application.routes.draw do
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
  get '/search_businesses', to: 'fetch#search_businesses'
  get '/search_events', to: 'fetch#search_events'
  get '/business_reviews', to: 'fetch#business_reviews'
end
```
