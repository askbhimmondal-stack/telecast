<!DOCTYPE html>
<html>
<head>
<title>Telecast</title>
<script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
</head>

<body style="background-color: #cae6cd;">
<center>
<div style="margin-top: 10px;">
  <img src="logo.png" alt="Logo" border="0" style="width: 200px; height: auto;">
</div>

<hr width="80%" style="border-color: #ccc;">

<video id="v" width="640" height="360" controls></video>

<br><br>

Search Channel:<br>
<input type="text" id="searchBox" placeholder="Type channel name..." 
       style="width:300px;padding:5px;font-size:16px;border:1px solid #ccc;">
<br>
<select id="ch" size="5" style="width:320px;display:none;margin-top:5px;border:1px solid #ccc;">
</select>

<br>
<input type="button" value="PLAY" onclick="playit()" style="font-size:16px;margin-top:5px;padding:5px 15px;background:#4CAF50;color:white;border:none;cursor:pointer;">

<br><br>
<span id="msg">Loading channels...</span>

<hr width="80%" style="border-color: #ccc;">
<medium>Made with ❤️ for Pooja</medium>
</center>

<script>
var M3U = "https://iptv-org.github.io/iptv/countries/in.m3u";
var video = document.getElementById('v');
var sel = document.getElementById('ch');
var searchBox = document.getElementById('searchBox');
var msg = document.getElementById('msg');
var hls = null;
var allChannels = [];

fetch(M3U)
.then(r => r.text())
.then(data => {
  var lines = data.split('\n');
  var name = '';
  for(var i=0; i<lines.length; i++){
    var line = lines[i].trim();
    if(line.startsWith('#EXTINF')){
      name = line.split(',').pop().trim();
    } else if(line.startsWith('http') && name){
      allChannels.push({
        name: name,
        url: line
      });
      name = '';
    }
  }
  msg.innerHTML = 'Channels loaded: ' + allChannels.length + '. Type to search.';
})
.catch(e => {
  msg.innerHTML = 'Error loading list.';
});

searchBox.addEventListener('input', function() {
  var searchText = this.value.toLowerCase().trim();
  var channelList = document.getElementById('ch');
  
  channelList.innerHTML = '';
  
  if (searchText.length < 2) {
    channelList.style.display = 'none';
    return;
  }
  
  var filteredChannels = allChannels.filter(function(channel) {
    return channel.name.toLowerCase().includes(searchText);
  });
  
  filteredChannels.forEach(function(channel) {
    var opt = document.createElement('option');
    opt.value = channel.url;
    opt.text = channel.name;
    channelList.add(opt);
  });
  
  if (filteredChannels.length > 0) {
    channelList.style.display = 'block';
    channelList.size = Math.min(filteredChannels.length, 8);
  } else {
    channelList.style.display = 'none';
    msg.innerHTML = 'No channels found.';
  }
});

sel.addEventListener('change', function() {
  if (this.value) {
    playit();
  }
});

document.addEventListener('click', function(event) {
  var channelList = document.getElementById('ch');
  if (event.target !== searchBox && event.target !== channelList) {
    channelList.style.display = 'none';
  }
});

function playit(){
  var url = sel.value;
  var selectedOption = sel.options[sel.selectedIndex];
  
  if(!url || !selectedOption){
    msg.innerHTML = 'Select a channel first.';
    return;
  }
  
  var name = selectedOption.text;
  msg.innerHTML = 'Now playing: ' + name;
  
  sel.style.display = 'none';
  
  if(hls){ hls.destroy(); hls=null; }
  video.pause();
  video.src='';

  if(Hls.isSupported()){
    hls = new Hls();
    hls.loadSource(url);
    hls.attachMedia(video);
    hls.on(Hls.Events.MANIFEST_PARSED,function(){ video.play(); });
  } else if(video.canPlayType('application/vnd.apple.mpegurl')){
    video.src = url;
    video.play();
  } else {
    msg.innerHTML = 'Browser not supported.';
  }
}
</script>
</body>
</html>
# telecast
