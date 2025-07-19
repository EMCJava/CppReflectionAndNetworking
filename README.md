# Cpp Reflection & Networking with Unreal Engine inspired Reflection System

Minimal working client server messaging system using frame synchronization and state synchronization

```C++

//==============================
//|    NetMessageSystem.hxx    |
//==============================

#if IS_Server
#    include <Network/GameServer.hxx>
#endif

#include <NetMessageSystem.generated.hxx>

#include <iostream>

SCLASS( )
// template <typename Ty> it can also be a template class
class CNetMessageSystem
{
    SCLASS_GENERATE_BODY( )

    void OnPlayerConnectionStatusChange( uint32_t HSteamNetConnection, bool IsReady );

public:
    CNetMessageSystem( );

    SFUNCTION( ServerMulticast )
    void BroadcastMessage( int PlayerID, std::string Message );

    SFUNCTION( ClientToServer )
    void UploadMessage( std::string Message );

protected:
    SPROPERTY( Replicated )
    std::array<std::string, MaxConnections> PlayerNames;
};

//==============================
//|    NetMessageSystem.cxx    |
//==============================

CNetMessageSystem::CNetMessageSystem( )
{
    RUN_ON_SERVER( CGameServer::AddOnPlayerStatusChangeCallback( std::bind_front( &CNetMessageSystem::OnPlayerConnectionStatusChange, this ) ) );
}

void
CNetMessageSystem::OnPlayerConnectionStatusChange( uint32_t HSteamNetConnection, bool IsReady )
{
#if IS_Server
    const auto* GameServer = CGameServer::TryGetInstance( );
    VERIFY( GameServer, return )
    const auto PlayerIndex = GameServer->ConnectionHandleToPlayerIndex( HSteamNetConnection );
    VERIFY( PlayerIndex < PlayerNames.size( ), return )
    PlayerNames[ PlayerIndex ] = IsReady ? GameServer->GetPlayerName( PlayerIndex ) : "";
#endif
}

void
CNetMessageSystem::BroadcastMessage_Impl( int PlayerID, std::string Message )
{
    std::cout << "Message from: " << PlayerNames[ PlayerID ] << " [" << Message << ']' << std::endl;
}

void
CNetMessageSystem::UploadMessage_Impl( uint32_t HSteamNetConnection, std::string Message )
{
#if IS_Server
    const auto* GameServer = CGameServer::TryGetInstance( );
    VERIFY( GameServer, return )
    const auto PlayerIndex = GameServer->ConnectionHandleToPlayerIndex( HSteamNetConnection );
    VERIFY( PlayerIndex < PlayerNames.size( ), return )
    BroadcastMessage( PlayerIndex, std::move( Message ) );
#endif
}
```
