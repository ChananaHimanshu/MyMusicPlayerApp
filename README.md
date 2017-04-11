# MyMusicPlayerApp
its a music player app for beginners.
package com.example.wittybrains.newmediaplayerapp;

import android.content.ContentResolver;
import android.database.Cursor;
import android.media.MediaPlayer;
import android.net.Uri;
import android.os.Handler;
import android.provider.MediaStore;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.ListView;
import android.widget.SeekBar;
import android.widget.TextView;
import android.widget.Toast;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.concurrent.TimeUnit;

public class MainActivity extends AppCompatActivity {

    private ListView listView;
    private ArrayAdapter adapter;
    private ArrayList<HashMap<String, String>> songsList;

    private HashMap<String, String> song;
    private String songIndex = "";

    private MediaPlayer mediaPlayer;
    private Handler myHandler = new Handler();

    private double startingTime = 0, endTime = 0;
    private int getForward = 2000, getBackward = 2000;
    private Button play_pause, forward, backward, next_song, previous_song;
    private TextView song_starting, song_name, song_ending;
    private SeekBar seekBar;
    private int count = 0;

    //String path="/home/wittybrains/Music/myMusic";
    //Uri filePath = Uri.parse(Environment.getExternalStorageDirectory() + path);
    //Uri filePath = Uri.parse(Environment.getExternalStorageDirectory() + "/myMusic");

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        getInitialised();
        getSongList();
        //setting the songs list on ListView
        adapter = new ArrayAdapter(MainActivity.this, android.R.layout.simple_list_item_1, songsList);
        listView.setAdapter(adapter);
        //mediaPlayer = MediaPlayer.create(MainActivity.this, R.raw.song1);


        //listener on listview
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, final View view, int position, long id) {
                //Toast.makeText(MainActivity.this, "U selected " + songsList.get(position), Toast.LENGTH_SHORT).show();
                songIndex = String.valueOf(songsList.get(position));
                String songTitle = songsList.get(position).get("songTitle");
                song_name.setText(songTitle);

                mediaPlayer.start();

                mediaPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
                    @Override
                    public void onCompletion(MediaPlayer mediaPlayer) {
                        Toast.makeText(MainActivity.this, "I'm Finished", Toast.LENGTH_SHORT).show();
                    }
                });

                play_pause.setBackgroundResource(R.drawable.pause);
                startingTime = mediaPlayer.getCurrentPosition();
                endTime = mediaPlayer.getDuration();
                song_ending.setText(String.format("%d min, %d sec", TimeUnit.MILLISECONDS.toMinutes((long) endTime),
                        TimeUnit.MILLISECONDS.toSeconds((long) endTime) -
                                TimeUnit.MINUTES.toSeconds((TimeUnit.MILLISECONDS.toMinutes((long) endTime)))));
                myHandler.postDelayed(UpdateData, 100);
            }
        });


        //listeners on Buttons
        play_pause.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                if (!(mediaPlayer.isPlaying())) {

                    songIndex = String.valueOf(songsList.get(0));
                    String songTitle = songsList.get(0).get("songTitle");
                    song_name.setText(songTitle);

                    mediaPlayer.start();
                    play_pause.setBackgroundResource(R.drawable.pause);

                    startingTime = mediaPlayer.getCurrentPosition();
                    endTime = mediaPlayer.getDuration();
                    song_ending.setText(String.format("%d min, %d sec", TimeUnit.MILLISECONDS.toMinutes((long) endTime),
                            TimeUnit.MILLISECONDS.toSeconds((long) endTime) -
                                    TimeUnit.MINUTES.toSeconds((TimeUnit.MILLISECONDS.toMinutes((long) endTime)))));
                    //myHandler.postDelayed(UpdateData, 100);

                } else if ((mediaPlayer.isPlaying())) {

                    mediaPlayer.pause();
                    play_pause.setBackgroundResource(R.drawable.play);

                }
            }
        });

        forward.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                int time = mediaPlayer.getCurrentPosition();
                if (time + getForward <= endTime) {
                    startingTime = time + getForward;
                    mediaPlayer.seekTo((int) (startingTime));
                    Toast.makeText(MainActivity.this, " Jumped 2 seconds forward ", Toast.LENGTH_SHORT).show();
                } else {
                    Toast.makeText(MainActivity.this, " Can't go beyond...", Toast.LENGTH_SHORT).show();
                }
            }
        });

        backward.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                int time = mediaPlayer.getCurrentPosition();
                if (time + getForward < endTime) {
                    startingTime = time - getBackward;
                    mediaPlayer.seekTo((int) startingTime);
                    Toast.makeText(MainActivity.this, " Jumped 2 second back ", Toast.LENGTH_SHORT).show();
                } else {
                    Toast.makeText(MainActivity.this, " Can't go before...", Toast.LENGTH_SHORT).show();
                }
            }
        });

        previous_song.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "<------- previous song clicked.", Toast.LENGTH_SHORT).show();

            }
        });

        next_song.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "next song clicked ------->", Toast.LENGTH_SHORT).show();

            }
        });

    }


    private void getInitialised() {
        listView = (ListView) findViewById(R.id.listView);
        play_pause = (Button) findViewById(R.id.play_pause);
        forward = (Button) findViewById(R.id.forward);
        backward = (Button) findViewById(R.id.backward);
        previous_song = (Button) findViewById(R.id.previous_song);
        next_song = (Button) findViewById(R.id.next_song);
        song_starting = (TextView) findViewById(R.id.textView2); //for song started
        song_ending = (TextView) findViewById(R.id.textView3); //for song remaining
        song_name = (TextView) findViewById(R.id.textView4); // song name
        seekBar = (SeekBar) findViewById(R.id.seekBar);
        seekBar.setMax((int) endTime);
        //initially the buttons are not enabled.,.,.,
        forward.setEnabled(false);
        backward.setEnabled(false);
        songsList = new ArrayList<HashMap<String, String>>();
        song = new HashMap<String, String>();
        mediaPlayer = new MediaPlayer();
    }


    private Runnable UpdateData = new Runnable() {
        @Override
        public void run() {
            startingTime = mediaPlayer.getCurrentPosition();
            song_starting.setText(String.format("%d min,\n %d sec", TimeUnit.MILLISECONDS.toMinutes((long) startingTime),
                    TimeUnit.MILLISECONDS.toSeconds((long) startingTime) -
                            TimeUnit.MINUTES.toSeconds((TimeUnit.MILLISECONDS.toMinutes((long) startingTime)))));
            seekBar.setMax((int) endTime);
            seekBar.setProgress((int) startingTime);
            myHandler.postDelayed(this, 100);
        }
    };


    //getting the list of songs
    public void getSongList() {
        ContentResolver contentResolver = getContentResolver();
        Uri musicUri = MediaStore.Audio.Media.INTERNAL_CONTENT_URI;
        // //Uri musicUri = MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
        //Uri musicUri = filePath;

        String selection = MediaStore.Audio.Media.IS_MUSIC + "!= 0";
        String sortOrder = MediaStore.Audio.Media.TITLE + " ASC";
        Cursor musicCursor = contentResolver.query(musicUri, null, selection, null, sortOrder);

        //Cursor musicCursor = contentResolver.query(musicUri, null, null, null, null);
        if (musicCursor != null && musicCursor.moveToFirst()) {
            int titleColumn = musicCursor.getColumnIndex(MediaStore.Audio.Media.TITLE);
            int idColumn = musicCursor.getColumnIndex(MediaStore.Audio.Media._ID);
            //int artistColumn = musicCursor.getColumnIndex(MediaStore.Audio.Media.ARTIST);
            do {
                long thisId = musicCursor.getLong(idColumn);
                String thisTitle = musicCursor.getString(titleColumn);
                //String thisArtist = musicCursor.getString(artistColumn);
                song.put("songTitle", thisTitle);
                //song.put("songArtist", thisArtist);
                song.put("songId", String.valueOf(thisId));
                songsList.add(song);
                count++;
            } while (musicCursor.moveToNext());
        }
        Toast.makeText(MainActivity.this, "Total songs fetched = " + count, Toast.LENGTH_LONG).show();
    }


}
