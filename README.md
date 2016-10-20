# Lesson03-material
###Το 3ο μάθημα περιλαμβάνει κλήση απομακρυσμένων λειτουργιών (Web Services) με σκοπό την λήψη των προβλέψεων για τις καιρικές συνθήκες απο πραγματικές υπηρεσίες καιρού.

Αρχικά θα πρέπει να εξοικειωθούμε με την κλήση απομακρυσμένων λειτουργιών.
Θα χρησιμοποιήσουμε το http://openweathermap.org/ για να πάρουμε πληροφορίες και προβλέψεις για τον καιρό στην περιοχή μας.
Για να λειτουργήσει χρειάζεται λογαριασμό και ειδικό κλειδί, οπότε καλό είναι να κάνουμε έναν. 
Προσωρινά μπορούμε να καλέσουμε τα Web Services με το API KEY 27949ea6b6dffa1dad1deb925c9b024b που ανήκει στο username *teohaik*
Σημείωση: Επιτρέπονται μόνο 60 κλήσεις ανά λεπτό.
 
Για να πάρουμε τις προβλέψεις για τις επόμενες 7 ημέρες για την πόλη της Θεσσαλονίκης θα πρέπει να καλέσουμε το εξής:

```
api.openweathermap.org/data/2.5/forecast/daily?id=734077&units=metric&cnt=7&APPID=27949ea6b6dffa1dad1deb925c9b024b
```

Τι σημαίνει αυτό το μακαρόνι;
Τι είναι το HTTP Request?    Δες εδώ: http://www.tutorialspoint.com/http/http_requests

Πώς θα καλέσουμε αυτό το Web Service μέσα απο τον κώδικά μας;

Ως προετοιμασία θα πούμε 2 λόγια το logging

![logging types](https://github.com/UomMobileDevelopment/Lesson03-material/blob/master/logging-types.png)


Θα χρειαστούμε αυτό το κομμάτι κώδικα:

```       
            // These two need to be declared outside the try/catch
            // so that they can be closed in the finally block.
            HttpURLConnection urlConnection = null;
            BufferedReader reader = null;

            // Will contain the raw JSON response as a string.
            String forecastJsonStr = null;

            try {
                // Construct the URL for the OpenWeatherMap query
                // Possible parameters are avaiable at OWM's forecast API page, at
                // http://openweathermap.org/API#forecast
                URL url = new URL("http://api.openweathermap.org/data/2.5/forecast/daily?q=94043&mode=json&units=metric&cnt=7");

                // Create the request to OpenWeatherMap, and open the connection
                urlConnection = (HttpURLConnection) url.openConnection();
                urlConnection.setRequestMethod("GET");
                urlConnection.connect();

                // Read the input stream into a String
                InputStream inputStream = urlConnection.getInputStream();
                StringBuffer buffer = new StringBuffer();
                if (inputStream == null) {
                    // Nothing to do.
                    return null;
                }
                reader = new BufferedReader(new InputStreamReader(inputStream));

                String line;
                while ((line = reader.readLine()) != null) {
                    // Since it's JSON, adding a newline isn't necessary (it won't affect parsing)
                    // But it does make debugging a *lot* easier if you print out the completed
                    // buffer for debugging.
                    buffer.append(line + "\n");
                }

                if (buffer.length() == 0) {
                    // Stream was empty.  No point in parsing.
                    return null;
                }
                forecastJsonStr = buffer.toString();
            } catch (IOException e) {
                Log.e("PlaceholderFragment", "Error ", e);
                // If the code didn't successfully get the weather data, there's no point in attemping
                // to parse it.
                return null;
            } finally{
                if (urlConnection != null) {
                    urlConnection.disconnect();
                }
                if (reader != null) {
                    try {
                        reader.close();
                    } catch (final IOException e) {
                        Log.e("PlaceholderFragment", "Error closing stream", e);
                    }
                }
            }

            return rootView;
        }
    }
}

```

μεταφέρουμε τον κώδικα μέσα στη μέθοδο ```onCreateView``` της κλάσης ```PlaceholderFragment``` μετά την κλήση:
```
listView.setAdapter(...)
```

Η εκτέλεση της εφαρμογής σε αυτό το σημείο θα βγάλει σφάλμα και θα κρασάρει την εφαρμογή. 
Το σφάλμα είναι του τύπου ```NetworkOnMainThreadException``` που μας λέει ότι απαγορεύεται να δημιουργούμε συνδέσεις δικτύου στη main και γενικότερα στο MainThread. 

Λίγα λόγια για τα Threads

![Threads](https://github.com/UomMobileDevelopment/Lesson03-material/blob/master/threads.png)


Θα χρειαστούμε μια κλάση η οποία απλοποιεί τη διαδικασία δημιουργίας Thread and UI thread synchronization 

##[AsyncTask](https://developer.android.com/reference/android/os/AsyncTask.html)##

Αφού μελετήσουμε την AsyncTask, ήρθε η ώρα να τη χρησιμοποιήσουμε για να μεταφέρουμε εκεί τον ανωτέρω κώδικα που επικολλήσαμε πρόχειρα μέσα στην PlaceholderFragment.

Ξεκινάμε με εργασίες αναδόμησης:

   1. Rename PlaceholderFragment -> ForecastFragment
   2. Move ForecastFragment to new file
   3. Create a new AsyncTask child called FetchWeatherTask with the networking code snippet from above
   

