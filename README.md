# PRODIGY_AD_03

package com.example.stopwatchapp

import android.os.Bundle
import android.os.Handler
import android.os.Looper
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.example.stopwatchapp.ui.theme.StopwatchappTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            StopwatchappTheme {
                StopwatchApp()
            }
        }
    }
}

@Composable
fun StopwatchApp() {
    var time by remember { mutableStateOf("00:00:000") }
    var isRunning by remember { mutableStateOf(false) }
    var startTime by remember { mutableStateOf(0L) }
    var elapsedTime by remember { mutableStateOf(0L) }

    val handler = remember { Handler(Looper.getMainLooper()) }

    val runnable = remember {
        object : Runnable {
            override fun run() {
                val current = System.currentTimeMillis()
                val total = elapsedTime + (current - startTime)
                val mins = (total / 60000).toInt()
                val secs = ((total % 60000) / 1000).toInt()
                val millis = (total % 1000).toInt()
                time = String.format("%02d:%02d:%03d", mins, secs, millis)
                handler.postDelayed(this, 10)
            }
        }
    }

    StopwatchUI(
        time = time,
        onStart = {
            if (!isRunning) {
                startTime = System.currentTimeMillis()
                handler.post(runnable)
                isRunning = true
            }
        },
        onPause = {
            if (isRunning) {
                elapsedTime += System.currentTimeMillis() - startTime
                handler.removeCallbacks(runnable)
                isRunning = false
            }
        },
        onReset = {
            handler.removeCallbacks(runnable)
            isRunning = false
            startTime = 0L
            elapsedTime = 0L
            time = "00:00:000"
        }
    )
}

@Composable
fun StopwatchUI(
    time: String,
    onStart: () -> Unit,
    onPause: () -> Unit,
    onReset: () -> Unit
) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(32.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = time,
            fontSize = 48.sp,
            fontWeight = FontWeight.Bold
        )
        Spacer(modifier = Modifier.height(32.dp))
        Row(
            horizontalArrangement = Arrangement.SpaceEvenly,
            modifier = Modifier.fillMaxWidth()
        ) {
            Button(onClick = onStart) {
                Text("Start")
            }
            Button(onClick = onPause) {
                Text("Pause")
            }
            Button(onClick = onReset) {
                Text("Reset")
            }
        }
    }
}
