# practical-
#include <iostream>
#include <map>
#include <vector>
#include <cmath>
#include <ctime>

using namespace std;

struct Transaction {
    double latitude;
    double longitude;
    time_t timestamp;
};

map<string, vector<Transaction> > recentTransactions;

const double MAX_TIME_DIFF_MINUTES = 5.0;
const double IMPOSSIBLE_SPEED_KMH = 1000.0;
const double EARTH_RADIUS_KM = 6371.0;

double toRadians(double degree) {
    return degree * M_PI / 180.0;
}

double haversine(double lat1, double lon1, double lat2, double lon2) {
    double dLat = toRadians(lat2 - lat1);
    double dLon = toRadians(lon2 - lon1);
    lat1 = toRadians(lat1);
    lat2 = toRadians(lat2);

    double a = sin(dLat / 2) * sin(dLat / 2) +
               cos(lat1) * cos(lat2) *
               sin(dLon / 2) * sin(dLon / 2);
    double c = 2 * asin(sqrt(a));
    return EARTH_RADIUS_KM * c;
}

bool isGeographicallyImpossible(const Transaction& t1, const Transaction& t2) {
    double distance = haversine(t1.latitude, t1.longitude, t2.latitude, t2.longitude);
    double timeDiffMinutes = difftime(t2.timestamp, t1.timestamp) / 60.0;
    if (timeDiffMinutes <= 0) return true;
    double speed = distance / (timeDiffMinutes / 60.0); // km/h
    return speed > IMPOSSIBLE_SPEED_KMH;
}

void blockCard(const string& accountId) {
    cout << "🚨 Fraud Alert: Card for account " << accountId << " has been BLOCKED.\n";
}

void processTransaction(const string& accountId, double latitude, double longitude) {
    time_t now = time(0);
    Transaction newTransaction = {latitude, longitude, now};

    if (recentTransactions.find(accountId) != recentTransactions.end()) {
        vector<Transaction>& txns = recentTransactions[accountId];
        for (size_t i = 0; i < txns.size(); ++i) {
            double timeDiff = difftime(now, txns[i].timestamp) / 60.0;
            if (timeDiff <= MAX_TIME_DIFF_MINUTES) {
                if (isGeographicallyImpossible(txns[i], newTransaction)) {
                    blockCard(accountId);
                    return;
                }
            }
        }
    }

    // Add transaction
    recentTransactions[accountId].push_back(newTransaction);

    // Remove old transactions
    vector<Transaction>& txns = recentTransactions[accountId];
    vector<Transaction> validTxns;
    for (size_t i = 0; i < txns.size(); ++i) {
        double age = difftime(now, txns[i].timestamp) / 60.0;
        if (age <= MAX_TIME_DIFF_MINUTES) {
            validTxns.push_back(txns[i]);
        }
    }
    recentTransactions[accountId] = validTxns;
}

// -------------------
int main() {
    string accountId = "user123";

    // Transaction in New York
    processTransaction(accountId, 40.7128, -74.0060);

    // Simulate manual wait (pretend 2 mins passed)
    // Wait for user input instead
    cout << "Press Enter to simulate next transaction (2 mins later)...\n";
    cin.get();

    // Simulate 2 minutes later by subtracting time in the past
    time_t now = time(0);
    time_t fakePast = now - (2 * 60); // 2 mins earlier

    // Overriding manually by reusing process logic
    Transaction london = {51.5074, -0.1278, fakePast};
    recentTransactions[accountId].push_back(london);

    // Now trigger current transaction again to test detection
    processTransaction(accountId, 51.5074, -0.1278);

    return 0;
}
