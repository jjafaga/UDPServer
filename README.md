# UDPServer
# UDP server code used to receive messages from a UDP client code
#include <iostream>
#include <ws2tcpip.h>

#pragma comment (lib, "ws2_32.lib")
using namespace std;

void main()
{
	const int twn = 1024, twe = 256;

	// Winsock Startup
	WSADATA data;
	WORD version = MAKEWORD(2, 2);
	int trigger = WSAStartup(version, &data);
	
	if (trigger != 0)
	{
		cout << "Can't start Winsock!" << trigger;
		return;
	}
	 
	// Socket Creation
	SOCKET in = socket(AF_INET, SOCK_DGRAM, 0);		// Data gram
	sockaddr_in serverHint;
	serverHint.sin_addr.S_un.S_addr = ADDR_ANY;
	serverHint.sin_family = AF_INET;
	serverHint.sin_port = htons(54000);	// Conversion from little to big endian
	
	// Binding Socket to IP address and port
	if (bind(in, (sockaddr*)&serverHint, sizeof(serverHint)) == SOCKET_ERROR)
	{
		cout << "Can't bind socket! " << WSAGetLastError() << endl;
		return;
	}

	sockaddr_in client;
	int clientLength = sizeof(client);
	ZeroMemory(&client, clientLength);	// client information that connects to the server

	char buff[twn]; 	// Message of the client

	// Enter Loop
	while (true)
	{
		ZeroMemory(buff, twn);	// buffer where message is recieved

		// Wait for message
		int messIn = recvfrom(in, buff, twn, 0, (sockaddr*)&client, &clientLength);		// &client has client information
		if (messIn == SOCKET_ERROR)
		{
			cout << "Error receiving from client!" << WSAGetLastError() << endl;
			continue;
		}

		// Display message
		char clientIP[twe];
		ZeroMemory(clientIP, twe);

		inet_ntop(AF_INET, &client.sin_addr, clientIP, twe);	// Take information from &client and convert to string

		cout << "Message received from " << clientIP << " : " << buff << endl;
	}

	// Close socket
	closesocket(in);

	// Winsock Shutdown
	WSACleanup();
}
