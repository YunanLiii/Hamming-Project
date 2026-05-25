#include <iostream>
#include <string>
#include <vector>
#include <random>
#include <stdexcept>
#include <iomanip>

using std::cout;
using std::cin;
using std::string;
using std::vector;

struct NoiseResult {
    string noisy;
    int flips;
};

char flip_bit(char bit) {
    if (bit == '0') return '1';
    if (bit == '1') return '0';
    throw std::invalid_argument("Bit must be '0' or '1'.");
}

void validate_binary(const string& bits) {
    if (bits.empty()) {
        throw std::invalid_argument("Input cannot be empty.");
    }
    for (char c : bits) {
        if (c != '0' && c != '1') {
            throw std::invalid_argument("Input must contain only 0 and 1.");
        }
    }
}

bool is_power_of_two(int x) {
    return x > 0 && (x & (x - 1)) == 0;
}

int required_parity_bits(int data_bits) {
    int r = 0;
    while ((1 << r) < data_bits + r + 1) {
        r++;
    }
    return r;
}

// Hamming code with even parity. Positions are counted from 1 on paper.
string encode_hamming(const string& data) {
    validate_binary(data);

    int m = static_cast<int>(data.size());
    int r = required_parity_bits(m);
    int n = m + r;

    string encoded(n, '0');

    // Fill data bits into non-parity positions.
    int data_index = 0;
    for (int pos = 1; pos <= n; pos++) {
        if (!is_power_of_two(pos)) {
            encoded[pos - 1] = data[data_index++];
        }
    }

    // Compute each parity bit.
    for (int p = 1; p <= n; p <<= 1) {
        int parity = 0;
        for (int pos = 1; pos <= n; pos++) {
            if (pos & p) {
                parity ^= (encoded[pos - 1] - '0');
            }
        }
        encoded[p - 1] = static_cast<char>('0' + parity);
    }

    return encoded;
}

int syndrome(const string& codeword) {
    validate_binary(codeword);

    int n = static_cast<int>(codeword.size());
    int error_position = 0;

    for (int p = 1; p <= n; p = p * 2) {
        int parity = 0;
        for (int pos = 1; pos <= n; pos++) {
            if (pos & p) {
                parity ^= (codeword[pos - 1] - '0');
            }
        }
        if (parity != 0) {
            error_position += p;
        }
    }

    return error_position;
}

string correct_hamming(const string& received, int& error_position) {
    validate_binary(received);

    string corrected = received;
    error_position = syndrome(received);

    if (error_position >= 1 && error_position <= static_cast<int>(corrected.size())) {
        corrected[error_position - 1] = flip_bit(corrected[error_position - 1]);
    }

    return corrected;
}

string extract_data(const string& codeword) {
    validate_binary(codeword);

    string data;
    for (int pos = 1; pos <= static_cast<int>(codeword.size()); pos++) {
        if (!is_power_of_two(pos)) {
            data.push_back(codeword[pos - 1]);
        }
    }
    return data;
}

NoiseResult add_noise(const string& encoded, double error_probability, std::mt19937& rng) {
    validate_binary(encoded);

    if (error_probability < 0.0 || error_probability > 1.0) {
        throw std::invalid_argument("Error probability must be between 0 and 1.");
    }

    std::bernoulli_distribution flip(error_probability);

    string noisy = encoded;
    int flips = 0;

    for (size_t i = 0; i < noisy.size(); i++) {
        if (flip(rng)) {
            noisy[i] = flip_bit(noisy[i]);
            flips++;
        }
    }
    

    return {noisy, flips};
}

void run_single_demo() {
    string data;
    double probability;

    cout << "Enter binary data, e.g. 1011: ";
    cin >> data;
    cout << "Enter bit flip probability from 0 to 1, e.g. 0.05: ";
    cin >> probability;

    std::random_device rd;
    std::mt19937 rng(rd());

    string encoded = encode_hamming(data);
    NoiseResult noise = add_noise(encoded, probability, rng);

    int error_position = 0;
    string corrected = correct_hamming(noise.noisy, error_position);
    string decoded = extract_data(corrected);

    cout << "\nOriginal data:    " << data << '\n';
    cout << "Encoded data:     " << encoded << '\n';
    cout << "Noisy received:   " << noise.noisy << '\n';
    cout << "Actual flips:     " << noise.flips << '\n';
    cout << "Syndrome result:  " << error_position;
    if (error_position == 0) cout << " meaning no error detected";
    else cout << " meaning position " << error_position << " is suspected";
    cout << '\n';
    cout << "Corrected code:   " << corrected << '\n';
    cout << "Decoded data:     " << decoded << '\n';
    cout << "Success?          " << (decoded == data ? "YES" : "NO") << "\n\n";
}

void run_statistics_demo() {
    string data;
    double probability;
    int trials;

    cout << "Enter binary data, e.g. 1011: ";
    cin >> data;
    cout << "Enter bit flip probability from 0 to 1, e.g. 0.05: ";
    cin >> probability;
    cout << "Enter number of trials, e.g. 10000: ";
    cin >> trials;

    if (trials <= 0) {
        throw std::invalid_argument("Trials must be positive.");
    }

    std::random_device rd;
    std::mt19937 rng(rd());

    string encoded = encode_hamming(data);
    int success = 0;
    int failure = 0;
    int zero_flip = 0;
    int one_flip = 0;
    int multi_flip = 0;

    for (int i = 0; i < trials; i++) {
        NoiseResult noise = add_noise(encoded, probability, rng);

        int error_position = 0;
        string corrected = correct_hamming(noise.noisy, error_position);
        string decoded = extract_data(corrected);

        if (decoded == data) success++;
        else failure++;

        if (noise.flips == 0) zero_flip++;
        else if (noise.flips == 1) one_flip++;
        else multi_flip++;
    }

    cout << "\nEncoded codeword length: " << encoded.size() << '\n';
    cout << "Trials:                  " << trials << '\n';
    cout << "Successes:               " << success << '\n';
    cout << "Failures:                " << failure << '\n';
    cout << std::fixed << std::setprecision(2);
    cout << "Success rate:            " << (100.0 * success / trials) << "%\n";
    cout << "Failure rate:            " << (100.0 * failure / trials) << "%\n";
    cout << "0 flips:                 " << zero_flip << '\n';
    cout << "1 flip:                  " << one_flip << '\n';
    cout << "2+ flips:                " << multi_flip << "  <-- normal Hamming code can fail here\n\n";
}

int main() {
    try {
        cout << "Hamming Code Error-Correction Simulator\n";
        cout << "1. Single encode/noise/correct demo\n";
        cout << "2. Statistics simulation\n";
        cout << "Choose option: ";

        int option;
        cin >> option;

        if (option == 1) {
            run_single_demo();
        } else if (option == 2) {
            run_statistics_demo();
        } else {
            cout << "Invalid option.\n";
        }
    } catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << '\n';
        return 1;
    }

    return 0;
}
