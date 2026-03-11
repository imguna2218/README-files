# SWE Role in using Skybrush : 

**Category 1 — The client wants data Skybrush doesn't store**

Skybrush shows live data on screen during the show. After the show ends, that data is gone. No history. No reports.

If client says "give me a report after every show showing which drones had issues" — that doesn't exist in Skybrush. Engineer builds it.

**Category 2 — The client wants Skybrush connected to something else**

Skybrush only knows about drones. If the client has a music system, LED screens, ticketing system, or anything else — Skybrush doesn't talk to them.

If client says "when the show ends, automatically trigger our LED screens to show our logo" — engineer writes the connection between Skybrush's API and the LED screen's API.

**Category 3 — The client wants Skybrush accessible from somewhere Skybrush doesn't support**

Skybrush Live runs on one laptop at the ground station. That's it.

If client says "I want to watch live drone status on my phone" or "I want my security team to be able to emergency land drones from their tablets" — that doesn't exist. Engineer builds a web app or mobile app that reads from Skybrush's API.

**Category 4 — The client wants safety rules Skybrush doesn't have**

Skybrush has basic safety. Battery low → come home. Signal lost → come home.

If client says "if wind speed exceeds 40 km/h, automatically pause the show and hold all drones in position" — Skybrush doesn't do this. Engineer connects a weather sensor's data to Skybrush's API and writes that rule.

**Category 5 — The client's drones have features Skybrush doesn't support**

Skybrush controls LED color. Basic colors. If the client's drones have special LED effects — strobe, pulse, rainbow — that the drone manufacturer built but Skybrush doesn't know about, engineer opens Skybrush source code and adds support for those commands.


## Category 6 — The client wants to run multiple shows simultaneously in different cities

**The situation:**

Your company is running 3 shows tonight. One in Hyderabad. One in Mumbai. One in Chennai. Each has its own laptop running Skybrush on-site.

Skybrush has no concept of multiple shows. Each installation is completely isolated. Your operations manager sitting in your office cannot see what is happening at any of the three shows right now.

**What the engineer builds:**

A central operations dashboard. Each on-site agent program pushes data to your cloud server. The dashboard shows all three shows on one screen simultaneously. Operations manager sees Hyderabad show is running fine, Mumbai show has 3 drones with low battery, Chennai show hasn't started yet. All from one screen in the office.

This is not a feature request from the client. This is an internal business need. Your company needs this to scale beyond one show at a time.

---

## Category 7 — The show must be repeatable with zero manual setup each time

**The situation:**

Your client is a theme park. They want the same drone show every Friday and Saturday night for one year. 104 shows. Same animation every time.

With plain Skybrush, before every show someone has to manually open Skybrush Live, check all drones, arm them, launch. After the show, manually confirm all landed, check logs manually.

104 times. Same steps. Every single time.

**What the engineer builds:**

An automated show runner. A scheduled program that at 8:55 PM every Friday and Saturday automatically runs pre-flight checks against Skybrush's API, confirms all drones are healthy, arms them, and at exactly 9:00 PM sends the launch command. After the show ends, automatically downloads logs, runs the health check, generates the report, and emails it to the client.

Zero human intervention for a routine show. Humans only get involved if the automated system raises a problem.

---

## Category 8 — The client wants the audience to interact with the drone show in real time

**The situation:**

The client is a music festival. They want the audience to vote on what shape the drones form next using their phones. "Vote for heart or vote for star." Winning shape appears in the sky within 30 seconds.

Skybrush has no concept of audience input. It follows a fixed pre-planned flight plan.

**What the engineer builds:**

A voting system. Audience opens a URL on their phone and votes. Your server counts votes in real time. When voting closes, your server takes the winning shape, generates a new flight plan segment for those drones using Skybrush's API, and sends updated waypoints to the drones mid-show.

This requires the engineer to deeply understand how Skybrush handles mid-show waypoint updates and whether the drones can safely transition from their current position to the new formation. This is not trivial. But it is possible with Skybrush's API.

---

## Category 9 — The client needs regulatory compliance documentation automatically

**The situation:**

In India, every drone show above 100 drones requires DGCA approval. The approval process requires submitting documents showing the exact flight path, maximum altitude, GPS boundary of the show, number of drones, operator credentials, and insurance details.

Preparing this manually for every show takes hours. You have to extract data from Skybrush Studio's flight plan, format it into government-required document formats, attach certifications, and submit.

Skybrush has no knowledge of DGCA or any regulatory body. It generates a flight plan file. That's it.

**What the engineer builds:**

A compliance document generator. It reads the flight plan file from Skybrush Studio, extracts all required data automatically — GPS boundaries, altitude limits, drone count, show duration — and fills it into the government-required document templates. Operator just reviews and submits. What took 3 hours takes 10 minutes.

As regulations change, the engineer updates the templates. One update fixes it for all future shows.

---

## Category 10 — The client's show must automatically adapt to real-time drone failures mid-show

**The situation:**

During the show, drone #203 loses GPS and lands itself. Now there is a visible gap in the formation. The audience sees a missing pixel in the shape.

Skybrush detects the failure and removes that drone from the show. But the remaining 999 drones keep flying their original positions. The gap stays visible for the rest of the show.

**What the engineer builds:**

A formation recovery system. It runs alongside Skybrush and monitors drone status via Skybrush's API. When it detects a drone failure, it calculates which neighbouring drones can slightly shift their positions to fill or reduce the gap without creating new collision risks. It sends updated waypoints to those specific drones via Skybrush's API within seconds.

The audience sees a very minor adjustment. The gap disappears. The show continues cleanly.

This is the hardest engineering problem in this entire list. It involves real-time geometry calculations, collision avoidance logic, and millisecond-level API calls. A designer cannot do this. A human operator cannot react fast enough. Only automated code can.
