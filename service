public enum ServiceError: String, Error {
    case apiError = "API Error"
    case invalidEndpoint = "Invalid Endpoint"
    case invalidResponse = "Invalid Response"
    case noData = "Response No Data"
    case decoderError = "Decoding Response Error"
    case encodingError = "Encoding Error"
    case encodingHeadersError = "Encoding Headers Error"
    case encodingParametersError = "Encoding Parameters Error"
}

public enum HTTPMethod: String{
    
    case get = "GET"
    case post = "POST"
    case put = "PUT"
    case patch = "PATCH"
    case delete = "DELETE"
}

class Services {
    var baseUrl = "https://api.rawg.io/api"
    
    typealias HTTPParameters = [String: Any]?
    typealias HTTPHeaders = [String: Any]?
    
    enum EncodingError: String, Error {
        case missingUrl = "Missing URLRequest"
        case headerNil = "Header nil"
    }
    
    func request<T: Codable>(endpoint: String, parameters: HTTPParameters, headers: HTTPHeaders, method: HTTPMethod = .get, body: Data?, completion: @escaping (Result<T, ServiceError>) -> ()) {
        guard let url = URL(string: baseUrl + endpoint) else {
            completion(.failure(.invalidEndpoint))
            return
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = method.rawValue
        request.httpBody = body
        
        do {
            try encodeParameters(urlRequest: &request, parameters: parameters)
        } catch {
            completion(.failure(.encodingParametersError))
        }
        
        do {
            try encodeHeaders(urlRequest: &request, headers: headers)
        } catch {
            completion(.failure(.encodingHeadersError))
        }
        
        let session = URLSession(configuration: .default)
        
        let task = session.dataTask(with: request) { data, response, error in
            guard error == nil else {
                completion(.failure(.apiError))
                return
            }
            
            guard let statusCode = (response as? HTTPURLResponse)?.statusCode, 200..<299 ~= statusCode else {
                completion(.failure(.invalidResponse))
                return
            }
            
            guard let data = data else {
                completion(.failure(.noData))
                return
            }
            
            do {
                let dataObject = try JSONDecoder().decode(T.self, from: data)
                DispatchQueue.main.async {
                    completion(.success(dataObject))
                }
            } catch {
                completion(.failure(.decoderError))
            }
            
        }
        task.resume()
        
    }
    
    private func encodeParameters(urlRequest: inout URLRequest, parameters: HTTPParameters) throws {
        if parameters == nil { return }
        
        guard let url = urlRequest.url, let parameters = parameters  else { throw EncodingError.missingUrl }
        
        if var urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: false), !parameters.isEmpty {
            urlComponents.queryItems = [URLQueryItem]()
            
            for (key,value) in parameters {
                let queryItem = URLQueryItem(name: key, value: "\(value)")
                
                urlComponents.queryItems?.append(queryItem)
            }
            
            urlRequest.url = urlComponents.url
        }
    }
    
    private func encodeHeaders(urlRequest: inout URLRequest, headers: HTTPHeaders) throws {
        if headers == nil { return }
        
        guard let headers = headers else { throw EncodingError.headerNil }
        
        for (key, value) in headers{
            urlRequest.setValue(value as? String, forHTTPHeaderField: key)
        }
    }
    
}
