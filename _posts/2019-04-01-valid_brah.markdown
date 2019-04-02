---
layout: post
title:      "Valid, brah?"
date:       2019-04-02 01:36:15 +0000
permalink:  valid_brah
---


Holy Crap that lab was hard! You know those failures in labs that you have! They keep itching at you, forcing you to not care about eating until you finish this and get through it! That was me these past two days and I am happy to say I learned a lot and hit a lot of hurdles on the way! Way more than I have ever had in my life, which was really awakening at how strong I have become as a developer and a person. I am always wanting myself for self- growth my persistency and consistency. I believe that Time and Effort are the only two essentials combined with persistency and consistency you can make quite amazing things happen in your life. One of them being: starting a bootcamp and going head first into becoming a programmer. I simply love being a programmer and the power that we have at our fingertips! I'm reading here and there on this website called [](https://medium.com/)! I actually understand what they are talking about, which in my case of being a person who can hated printing on computers to becoming a programmer I absolutely love my story and my life! I wanted to show you the code that made  me realize validations can be really powerful:

`class Song < ApplicationRecord
    validates :title, presence: true
    validates :title, uniqueness: true
    validates :released, inclusion: {in: [true, false]}
    validates :release_year, numericality: {less_than: Time.now.year}, if: :release?

    def release?
        released == true
    end
end
`

This right here is my second Custom Validation in my life which I am very proud of due to the fact that this code can be quite difficult to solve! I used my Custom Validation to be only check if the release_year is valid in the circumstance that `released: true` so that it would be okay with nil due to the fact that there was no `release_year` since the song has not been released. I struggled for an hour before I found the solution and was very close to getting up to go drive to get some pizza (actually part of my meal plan, lots of carbs for this week :) for all you health fitness peeps) and I finally found that the Custom Validation was needed. I had thought about `with_option` and doing some other very complexing things but in due time with enough hammering at the stubborn rock it came loose in my mind and I was able to plug the failure and pass everything! I hoped this helped somebody and gives any newbie in programming that you are not alone and to any expert in the field you can reminisce back in your early years when this was hard to you!

