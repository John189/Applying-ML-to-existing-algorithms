# Applying-ML-to-existing-algorithms
How to integrate a machine learning layer into existing algorithms

This repository is a description of how I integratde a python machine learning model as an extra layer of decision making in a strategy that runs in Ninjatrader. I'm not providing my Ninjatrader algorithms or my machine learning algorithms, just the basic concept of how they can be connected.

My Ninjatrader strategy runs on the 1 minute time timefame and will take longs and shorts and is successful but I feel it could be refined more with something like along the lines of a trend indicator. However I had no success with traditional trend indicators. 

I successfully increased the strategies profit factor  when I combined a machine learning prediction about the 15 minute chart direction. I used a linear regression/KNN model to predict the major trend direction changes and wrote the prediction to a .csv file. The ninjatrader strategy checks the .csv file as part of it's entry conditions and only enters in the ML prediction direction.

The code is simple and only provided as an example of how to quickly integrate ML decisions into already succesful strategies to boost the profit factor.

I wrote the Python ML model in Juypter Notebook. Once the model is working, it can be scheduled to run every 15 minutes using jupyter scheduler custom schedule with the following cron expression: */15 * * * MON-FRI

Inital ML model code setup looks like this:

    import pandas as pd
    import numpy as np
    import pandas_ta as ta
    import joblib 
    from scipy.stats import linregress
    import time
    
    time.sleep(5)
    
    # Path to your CSV file and model
    csv_file_path = r'C:\Users\X\Documents\NinjaTrader 8\OHLCVData.csv'
    model_file_path = r'C:\Users\X\Documents\NQDataML\NQ15m100p48b.sav'
    prediction_file_path = r'C:\Users\X\Documents\NQDataML\Predictions\prediction100p48b.csv'
    loaded_model = joblib.load(model_file_path)
    
    df = pd.read_csv(csv_file_path)
    
    # Your models code goes here

    # Make prediction
ModelPrediction = loaded_model.predict(X_model)

# Interpret prediction
print(f"Model Prediction: {ModelPrediction}")

import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# Check if the prediction is 1 or 2
if ModelPrediction in [1, 2]:

    with open(prediction_file_path, 'w') as file:
        file.write(f"Model Prediction: {ModelPrediction}\n")

    # Gmail credentials
    gmail_user =   # replace with your email
    gmail_password =   # replace with your password
    
    # Email setup
    sent_from = gmail_user
    to = ['@gmail.com']  # replace with the recipient's email
    # Customize the subject based on the prediction
    if ModelPrediction == 1:
        subject = 'Model 100p48b: Down'
    elif ModelPrediction == 2:
        subject = 'Model 100p48b : Up'
    body = f"The prediction from the model is: {ModelPrediction}"
    
    # Create MIME message
    msg = MIMEMultipart()
    msg['From'] = sent_from
    msg['To'] = ", ".join(to)
    msg['Subject'] = subject
    msg.attach(MIMEText(body, 'plain'))
    
    # Send the email
    try:
        server = smtplib.SMTP_SSL('smtp.gmail.com', 465)
        server.ehlo()
        server.login(gmail_user, gmail_password)
        server.sendmail(sent_from, to, msg.as_string())
        server.close()
    
        print("Email sent successfully!")
    except Exception as e:
        print(f"Something went wrong... {e}")

The email part is not necessary, but as the ML model essentially acts as the "on" switch for my code everyday, I like to get an email.

In my Ninjatrader C# strategy, I add the following pieces of code

    
  private string filePath = @"C:\Users\X\Documents\NQDataML\updated_df_model15m.csv";
		int prediction;
		private Dictionary<DateTime, int> predictions = new Dictionary<DateTime, int>();
  private int lastPrediction = 0;
  private DateTime lastPredictionTime;
		int currentPrediction;

  private void LoadPredictionsFromFile()
        {
            if (!File.Exists(filePath))
            {
                Print("CSV file not found");
                Log("CSV file not found", LogLevel.Error);
                return;
            }

            string[] lines = File.ReadAllLines(filePath);
			Print("Number of lines in file: " + lines.Length);
            foreach (string line in lines)
            {
                string[] columns = line.Split(',');
                if (columns.Length < 2)
                {
                    Print("Invalid line format, expected at least 2 columns but found: " + columns.Length + " in line: " + line);
                    continue;
                }

                DateTime entryTime;
				if (!DateTime.TryParseExact(columns[0], "yyyyMMdd HHmmss", CultureInfo.InvariantCulture, DateTimeStyles.None, out entryTime))
                {
                    Print("Failed to parse datetime: " + columns[0]);
                    continue;
                }

                int predictionValue;
				if (!int.TryParse(columns[1], out predictionValue))
                {
                    Print("Failed to parse prediction: " + columns[1]);
                    continue;
                }

                predictions[entryTime] = predictionValue;
				Print("Loaded prediction: Time - " + entryTime.ToString("yyyyMMdd HHmmss") + ", Value - " + predictionValue);
            }
        }
		
		private int GetPredictionForCurrentTime()
        {
            DateTime currentTime = Time[0];

            // Round down to the nearest 15 minutes
            DateTime roundedTime = new DateTime(currentTime.Year, currentTime.Month, currentTime.Day, currentTime.Hour, currentTime.Minute / 15 * 15, 0);

            
			if (predictions.TryGetValue(roundedTime, out currentPrediction))
            {
                lastPrediction = currentPrediction;
                lastPredictionTime = roundedTime;
            }
            else if (roundedTime > lastPredictionTime)
            {
                // If no new prediction is found and the current time has moved beyond the last prediction time, reset the prediction
                lastPrediction = 0;
            }

            return lastPrediction;
        }

        protected override void OnBarUpdate()
		{
		    if (CurrentBar < 21)
		        return;
			
			prediction = GetPredictionForCurrentTime();
			Print("Current prediction: " + prediction);



# Include the prediction in your entry conditions

	  if (your short entry conditions && currentPrediction == 1)
							{
	       // Enter the short trade
	       shortOrder = EnterShortLimit
	       }
	
	  if (your long entry conditions && currentPrediction == 2)
							{
	       // Enter the long trade
	       longOrder = EnterLongLimit
	       }

 
