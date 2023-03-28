# SignalR in Blazor
Real-time chat with Blazor Server SignalR

1. **Create a Blazor Server App**
2.  **Add a SignalR Hub**
- Add NuGet Package: `Microsoft.AspNetCore.SignalR.Client` 
- Add new folder to the solution, `Hub`. Create class `ChatHub.cs` for Sending Messages to the client.

 [ChatApp](https://github.com/Shalini-lodhi/ChatApp)/[ChatApp](https://github.com/Shalini-lodhi/ChatApp/tree/master/ChatApp)/[Hubs](https://github.com/Shalini-lodhi/ChatApp/tree/master/ChatApp/Hubs)/**ChatHub.cs**
```c#
using Microsoft.AspNetCore.SignalR;

namespace ChatApp.Hubs
{
    public class ChatHub : Hub
    {
        public async Task SendMessage(string user, string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", user, message);
        }
    }
}
```
3. **Configure `Program.cs`**
[ChatApp](https://github.com/Shalini-lodhi/ChatApp)/[ChatApp](https://github.com/Shalini-lodhi/ChatApp/tree/master/ChatApp)/**Program.cs**
```c#
using ChatApp.Data;
using ChatApp.Hubs;
using Microsoft.AspNetCore.ResponseCompression;

internal class Program
{
    private static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        // Add services to the container.
        builder.Services.AddRazorPages();
        builder.Services.AddServerSideBlazor();
        builder.Services.AddSingleton<WeatherForecastService>();
        //compressing the messages
        builder.Services.AddResponseCompression(opts =>
        {
            opts.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(new[] { "application/octet-stream" });
        });

        var app = builder.Build();

        // Configure the HTTP request pipeline.
        if (!app.Environment.IsDevelopment())
        {
            app.UseExceptionHandler("/Error");
        }

        app.UseStaticFiles();

        app.UseRouting();

        app.MapBlazorHub();
        app.MapHub<ChatHub>("/chathub");
        app.MapFallbackToPage("/_Host");

        app.Run();
    }
}
```
4.  **Add UserMessage Model**
[ChatApp](https://github.com/Shalini-lodhi/ChatApp)/[ChatApp](https://github.com/Shalini-lodhi/ChatApp/tree/master/ChatApp)/[Models](https://github.com/Shalini-lodhi/ChatApp/tree/master/ChatApp/Models)/**UserMessage.cs**
```c#
namespace ChatApp.Models
{
    public class UserMessage
    {
        public string UserName { get; set; }
        public string Message { get; set; }
        public bool CurrentUser { get; set; }
        public DateTime DataSent { get; set; }
    }
}
```
5. **Setup the Chat component**
[ChatApp](https://github.com/Shalini-lodhi/ChatApp)/[ChatApp](https://github.com/Shalini-lodhi/ChatApp/tree/master/ChatApp)/[Pages](https://github.com/Shalini-lodhi/ChatApp/tree/master/ChatApp/Pages)/**Index.razor**
```c#
@page "/"
@using Microsoft.AspNetCore.SignalR.Client
@using Models
@inject NavigationManager NavigationManager
@implements IAsyncDisposable
```
Messaging Area
```c#
<div class="container overflow-auto shadow-sm p-3 mb-5 bg-white rounded" style="height: 500px;">
    @if (!userMessages.Any())
    {
        <p>No messages yet, start chatting!</p>
    }

    @foreach (var userMessage in userMessages)
    {
        <div class="row mb-3 d-flex @(userMessage.CurrentUser ? "justify-content-end" : "")">
            <div class="card shadow @(userMessage.CurrentUser ? "color-green mr-5" : "ml-5")" style="width: 18rem;">
                <div class="card-header">
                    @(userMessage.CurrentUser ? "You" : userMessage.Username)
                </div>
                <ul class="list-group list-group-flush">
                    <li class="list-group-item @(userMessage.CurrentUser ? "color-green" : "")">@userMessage.Message</li>
                </ul>
                <div class="card-footer">
                    <span class="small">@userMessage.DateSent.ToString("HH:mm | MMM dd")</span>
                </div>
            </div>
        </div>
    }
</div>
```
Texting Area
```c#
<div class="container">
    <div class="row">
        <div class="col-3">
            <input @bind="usernameInput" type="text" class="form-control" placeholder="Your name" readonly="@isUserReadonly"/>
        </div>
        <div class="col-6">
            <textarea @bind="messageInput" class="form-control" placeholder="Start typing..."></textarea>
        </div>
        <div class="col-3">
            <button type="button" @onclick="Send" disabled="@(!IsConnected)" class="btn btn-primary">Send</button>
        </div>
    </div>
</div>
```
Functional Property
```c#
@code{
    private HubConnection hubConnection;
    private List<UserMessage> userMessages = new();
    private string usernameInput;
    private string messageInput;
    private bool isUserReadonly = false;

    public bool IsConnected => hubConnection.State == HubConnectionState.Connected;

    protected override async Task OnInitializedAsync()
    {
        hubConnection = new HubConnectionBuilder()
            .WithUrl(NavigationManager.ToAbsoluteUri("/chathub"))
            .Build();
        hubConnection.On<string, string>("ReceiveMessage", (user, message) =>
        {
            userMessages.Add(new UserMessage { Username = user, Message = message, CurrentUser = user == usernameInput, DateSent = DateTime.Now });
            StateHasChanged();
        });
        await hubConnection.StartAsync();
    }
    private async Task Send()
    {
        if (!string.IsNullOrEmpty(usernameInput) && !string.IsNullOrEmpty(messageInput))
        {
            await hubConnection.SendAsync("SendMessage", usernameInput, messageInput);
            isUserReadonly = true;
            messageInput = string.Empty;
        }
    }
    public async ValueTask DisposeAsync()
    {
        if (hubConnection is not null)
        {
            await hubConnection.DisposeAsync();
        }
    }
}
```
6. **Adding Color to the text**
[ChatApp](https://github.com/Shalini-lodhi/ChatApp)/[ChatApp](https://github.com/Shalini-lodhi/ChatApp/tree/master/ChatApp)/[wwwroot](https://github.com/Shalini-lodhi/ChatApp/tree/master/ChatApp/wwwroot)/[css](https://github.com/Shalini-lodhi/ChatApp/tree/master/ChatApp/wwwroot/css)/**site.css**
```css
.color-green {
background-color: #5CDB94;
}
```

## Output BlazorUI
<img width="956" alt="image" src="https://user-images.githubusercontent.com/55933789/228143026-11b08ce7-dc00-4398-a6e4-f97270b6cc10.png">
