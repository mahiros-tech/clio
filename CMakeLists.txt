cmake_minimum_required(VERSION 3.16.3)

project(clio)

option(BUILD_TESTS "Build tests" TRUE)

option(VERBOSE "Verbose build" TRUE)
if(VERBOSE)
  set(CMAKE_VERBOSE_MAKEFILE TRUE)
  set(FETCHCONTENT_QUIET FALSE CACHE STRING "Verbose FetchContent()")
endif()

if(NOT GIT_COMMIT_HASH)
  if(VERBOSE)
    message(WARNING "GIT_COMMIT_HASH not provided...looking for git")
  endif()
  find_package(Git)
  if(Git_FOUND)
      execute_process(COMMAND ${GIT_EXECUTABLE} describe --always --abbrev=8
          OUTPUT_STRIP_TRAILING_WHITESPACE OUTPUT_VARIABLE gch)
      if(gch)
          set(GIT_COMMIT_HASH "${gch}")
          message(STATUS "Git commit: ${GIT_COMMIT_HASH}")
          add_definitions(-DCLIO_GIT_COMMIT_HASH="${GIT_COMMIT_HASH}")
      endif()
  endif()
endif() #git

add_library(clio)
target_compile_features(clio PUBLIC cxx_std_20)
target_include_directories(clio PUBLIC src)

include(FetchContent)
include(ExternalProject)
include(CMake/settings.cmake)
include(CMake/ClioVersion.cmake)
include(CMake/deps/rippled.cmake)
include(CMake/deps/Boost.cmake)
include(CMake/deps/cassandra.cmake)
include(CMake/deps/Postgres.cmake)

target_sources(clio PRIVATE
  ## Main
  src/main/impl/Build.cpp
  ## Backend
  src/backend/BackendInterface.cpp
  src/backend/CassandraBackend.cpp
  src/backend/Pg.cpp
  src/backend/PostgresBackend.cpp
  src/backend/SimpleCache.cpp
  ## ETL
  src/etl/ETLSource.cpp
  src/etl/NFTHelpers.cpp
  src/etl/ReportingETL.cpp
  ## Subscriptions
  src/subscriptions/SubscriptionManager.cpp
  ## RPC
  src/rpc/RPC.cpp
  src/rpc/RPCHelpers.cpp
  src/rpc/Counters.cpp
  src/rpc/WorkQueue.cpp
  ## RPC Methods
  # Account
  src/rpc/handlers/AccountChannels.cpp
  src/rpc/handlers/AccountCurrencies.cpp
  src/rpc/handlers/AccountInfo.cpp
  src/rpc/handlers/AccountLines.cpp
  src/rpc/handlers/AccountOffers.cpp
  src/rpc/handlers/AccountObjects.cpp
  src/rpc/handlers/GatewayBalances.cpp
  src/rpc/handlers/NoRippleCheck.cpp
  # NFT
  src/rpc/handlers/NFTInfo.cpp
  # Ledger
  src/rpc/handlers/Ledger.cpp
  src/rpc/handlers/LedgerData.cpp
  src/rpc/handlers/LedgerEntry.cpp
  src/rpc/handlers/LedgerRange.cpp
  # Transaction
  src/rpc/handlers/Tx.cpp
  src/rpc/handlers/TransactionEntry.cpp
  src/rpc/handlers/AccountTx.cpp
  # Dex
  src/rpc/handlers/BookOffers.cpp
  # NFT
  src/rpc/handlers/NFTOffers.cpp
  # Payment Channel
  src/rpc/handlers/ChannelAuthorize.cpp
  src/rpc/handlers/ChannelVerify.cpp
  # Subscribe
  src/rpc/handlers/Subscribe.cpp
  # Server
  src/rpc/handlers/ServerInfo.cpp
  # Utility
  src/rpc/handlers/Random.cpp)

add_executable(clio_server src/main/main.cpp)
target_link_libraries(clio_server PUBLIC clio)

if(BUILD_TESTS)
  add_executable(clio_tests unittests/main.cpp)
  include(CMake/deps/gtest.cmake)
endif()

include(CMake/install/install.cmake)
if(PACKAGING)
    include(CMake/packaging.cmake)
endif()
